<script>
	import { onMount } from 'svelte';
	import {
		getFirestore,
		collection,
		onSnapshot,
		query,
		orderBy,
		updateDoc,
		doc,
		getDoc
	} from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';
	import { getStorage, ref, uploadBytes, getDownloadURL } from 'firebase/storage';
	import { getAuth } from 'firebase/auth';
	import { userStore } from '$lib/stores';

	export let data;
	let db = null;
	let storage = null;
	let homework = null;
	let isLoaded = false;

	onMount(async () => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);
		db = getFirestore(app);
		storage = getStorage(app);

		// Fetch the specific homework document using homeworkId
		let document = data.link;
		const homeworkRef = doc(db, 'homework', document);
		const homeworkSnapshot = await getDoc(homeworkRef);

		if (homeworkSnapshot.exists()) {
			const homeworkData = homeworkSnapshot.data();
			const completedBy = homeworkData.completedBy || []; // Initialize completedBy as an empty array if undefined
			homework = {
				id: homeworkSnapshot.id,
				...homeworkData,
				completedBy: Array.isArray(completedBy) ? completedBy : [] // Ensure completedBy is an array
			};
		}

		isLoaded = true;
	});

	async function uploadFile(file) {
		try {
			const storageRef = ref(storage, `homework/${file.name}`);
			await uploadBytes(storageRef, file);
			const downloadURL = await getDownloadURL(storageRef);
			return downloadURL;
		} catch (error) {
			console.error('Error uploading file:', error);
			throw error;
		}
	}

	async function submitHomework(event) {
		event.preventDefault();

		const { file } = event.target.elements;
		const fileURL = await uploadFile(file.files[0]);

		// Get the authenticated user
		const user = $userStore;
		console.log(user);

		if (user && homework) {
			const homeworkId = homework.id;
			try {
				// Update the user's homework collection
				const userHomeworkRef = doc(db, 'users', user.uid, 'homework', homeworkId);
				await updateDoc(userHomeworkRef, {
					completed: true,
					fileURL: fileURL
				});

				// Update the completedBy field in the homework document
				const homeworkRef = doc(db, 'homework', homeworkId);
				await updateDoc(homeworkRef, {
					completedBy: [...homework.completedBy, user.uid]
				});

				console.log('Homework submitted successfully.');
			} catch (error) {
				console.error('Error submitting homework:', error);
			}
		}

		// Reset the form after successful submission
		event.target.reset();
	}
</script>

<main>
	<h2>Homework Submission</h2>
	{#if isLoaded && homework}
		<form on:submit={submitHomework}>
			<label>
				File:
				<input type="file" name="file" accept="image/*" required />
			</label>
			<button type="submit">Submit Homework</button>
		</form>
	{:else}
		<p>Loading homework...</p>
	{/if}
</main>
