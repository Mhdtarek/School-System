<script>
	import { onMount } from 'svelte';
	import { getAuth, onAuthStateChanged } from 'firebase/auth';
	import { getFirestore, collection, addDoc } from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';
	import { getStorage, ref, uploadBytesResumable, getDownloadURL } from 'firebase/storage';

	let auth = null;
	let db = null;
	let storage = null;
	let currentUser = null;
	let homeworkName = '';
	let task = '';
	let dueDate = '';
	let file = null;
	let selectedClass = '';
	const classOptions = [1, 2, 3, 4, 5];

	onMount(() => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);

		// Check if the user is authenticated
		auth = getAuth(app);
		db = getFirestore(app);
		storage = getStorage(app);
		onAuthStateChanged(auth, (user) => {
			if (user) {
				currentUser = user;
			}
		});
	});

	async function uploadHomework() {
		if (!homeworkName || !task || !dueDate || !file || !selectedClass) {
			console.error('Please provide all the required information.');
			return;
		}

		try {
			// Upload the file to the Cloud Storage
			const storageRef = ref(storage, `homework/${file.name}`);
			const uploadTask = uploadBytesResumable(storageRef, file);

			// Get the download URL of the uploaded file
			const snapshot = await uploadTask;
			const downloadURL = await getDownloadURL(snapshot.ref);

			// Add a document to the "homework" collection in Firestore
			const homeworkData = {
				homeworkName,
				task,
				dueDate,
				completedBy: [],
				fileURL: downloadURL,
				class: selectedClass
			};

			const docRef = await addDoc(collection(db, 'homework'), homeworkData);
			console.log('Homework added successfully:', docRef.id);

			// Clear the form fields
			homeworkName = '';
			task = '';
			dueDate = '';
			file = null;
			selectedClass = '';
		} catch (error) {
			console.error('Error uploading homework:', error);
		}
	}
</script>

<main>
	<h2>Upload Homework</h2>
	<form on:submit|preventDefault={uploadHomework}>
		<label>
			Homework Name:
			<input type="text" bind:value={homeworkName} required />
		</label>
		<label>
			Task:
			<textarea bind:value={task} rows="4" required />
		</label>
		<label>
			Due Date:
			<input type="date" bind:value={dueDate} required />
		</label>
		<label>
			Class:
			<select bind:value={selectedClass} required>
				<option value="">Select Class</option>
				{#each classOptions as option}
					<option value={option}>{option}</option>
				{/each}
			</select>
		</label>
		<label>
			File:
			<input type="file" on:change={(event) => (file = event.target.files[0])} required />
		</label>
		<button type="submit">Upload Homework</button>
	</form>
</main>
