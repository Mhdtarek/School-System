<script>
	import { onMount } from 'svelte';
	import { getAuth, onAuthStateChanged } from 'firebase/auth';
	import {
		getFirestore,
		collection,
		getDocs,
		query,
		where,
		updateDoc,
		doc,
		getDoc
	} from 'firebase/firestore';
	import { goto } from '$app/navigation';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';

	let auth = null;
	let db = null;
	let unverifiedUsers = [];

	onMount(() => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);

		// Check if the user is authenticated and is an admin
		auth = getAuth(app);
		db = getFirestore(app);
		onAuthStateChanged(auth, async (user) => {
			if (user && user.uid) {
				const userDocRef = doc(db, 'users', user.uid);
				const userDocSnapshot = await getDoc(userDocRef);
				if (userDocSnapshot.exists() && userDocSnapshot.data().role === 'admin') {
					// User is authenticated and is an admin
					// Retrieve the list of unverified users
					const unverifiedUsersQuery = query(
						collection(db, 'users'),
						where('verified', '==', false)
					);
					const unverifiedUsersSnapshot = await getDocs(unverifiedUsersQuery);
					unverifiedUsers = unverifiedUsersSnapshot.docs.map((doc) => ({
						uid: doc.id,
						...doc.data()
					}));
				} else {
					// User is not an admin or not authenticated
					goto('/');
				}
			} else {
				// User is not authenticated
				goto('/');
			}
		});
	});

	async function verifyUser(uid, role, selectedClass) {
		const userDocRef = doc(db, 'users', uid);
		try {
			await updateDoc(userDocRef, { verified: true, role: role, class: selectedClass });
			console.log('User verified successfully.');
			// Remove the user from the list by filtering it out
			unverifiedUsers = unverifiedUsers.filter((user) => user.uid !== uid);
			alert('User has been verified.');
		} catch (error) {
			console.error('Error verifying user:', error);
		}
	}
</script>

<main class="container">
	<table>
		<thead>
			<tr>
				<th>Name</th>
				<th>Email</th>
				<th>Role</th>
				<th>Class</th>
				<th>Action</th>
			</tr>
		</thead>
		<tbody>
			{#each unverifiedUsers as user}
				<tr key={user.uid}>
					<td>{user.displayName}</td>
					<td>{user.email}</td>
					<td>
						<select bind:value={user.role}>
							<option value="teacher">Teacher</option>
							<option value="student">Student</option>
							<option value="admin">Admin</option>
							<option value="parent">Parent</option>
						</select>
					</td>
					<td>
						<select bind:value={user.class}>
							<option value="1">1</option>
							<option value="2">2</option>
							<option value="3">3</option>
							<option value="4">4</option>
							<option value="5">5</option>
						</select>
					</td>
					<td>
						<button on:click={() => verifyUser(user.uid, user.role, user.class)}>Verify User</button
						>
					</td>
				</tr>
			{/each}
		</tbody>
	</table>
</main>
