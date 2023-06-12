<script>
	import { onMount, onDestroy } from 'svelte';
	import { initializeApp } from 'firebase/app';
	import {
		getAuth,
		signInWithPopup,
		GoogleAuthProvider,
		setPersistence,
		browserLocalPersistence,
		onAuthStateChanged
	} from 'firebase/auth';
	import { getFirestore, doc, setDoc, getDoc, collection, addDoc } from 'firebase/firestore';
	import { firebaseConfig } from '$lib/config.js';
	import { writable } from 'svelte/store';
	import { goto } from '$app/navigation';
	import { userStore } from '$lib/stores.js';

	let app = null;
	let auth = null;
	let db = null;

	async function initializeFirebase() {
		app = initializeApp(firebaseConfig);
		auth = getAuth(app);
		setPersistence(auth, browserLocalPersistence);
		db = getFirestore(app);
		onAuthStateChanged(auth, async (loggedInUser) => {
			if (loggedInUser) {
				try {
					const userDocRef = doc(db, 'users', loggedInUser.uid);
					const userDocSnapshot = await getDoc(userDocRef);
					if (!userDocSnapshot.exists()) {
						const userData = {
							displayName: loggedInUser.displayName,
							email: loggedInUser.email,
							photoURL: loggedInUser.photoURL,
							verified: false, // Set the "verified" property to false initially
							role: '', // Initialize the "role" property as empty
							class: 0 // Set the "class" property with a specific class number, e.g., 1
						};
						await setDoc(userDocRef, userData);
						userStore.set(userData);
					} else {
						userStore.set(userDocSnapshot.data());
					}
					goto('/');
				} catch (error) {
					console.error('Error retrieving user document:', error);
					goto('/');
				}
			} else {
				userStore.set({});
			}
		});
	}

	onMount(() => {
		initializeFirebase();
	});

	onDestroy(() => {
		app = null;
		auth = null;
		db = null;
	});

	async function signInWithGoogle() {
		const provider = new GoogleAuthProvider();

		try {
			const result = await signInWithPopup(auth, provider);
			const user = result.user;
			const userDocRef = doc(db, 'users', user.uid);
			const userDocSnapshot = await getDoc(userDocRef);

			if (!userDocSnapshot.exists()) {
				const userData = {
					displayName: user.displayName,
					email: user.email,
					photoURL: user.photoURL,
					verified: false, // Set the "verified" property to false initially
					role: '', // Initialize the "role" property as empty
					class: 1 // Set the "class" property with a specific class number, e.g., 1
				};

				await setDoc(userDocRef, userData);
			}

			// Create a "homework" collection for the user
			const homeworkCollectionRef = collection(db, 'users', user.uid, 'homework');

			// Track homework by adding documents to the collection
			await addDoc(homeworkCollectionRef, {
				// Add properties to track the user's homework
			});

			userStore.set(user);

			console.log('User signed in:', user);
		} catch (error) {
			console.error('Error signing in with Google:', error);
		}
	}
</script>

<main>
	<button on:click={signInWithGoogle}>Sign in with Google</button>
</main>
