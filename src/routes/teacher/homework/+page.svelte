<script>
	import { onMount } from 'svelte';
	import {
		getFirestore,
		collection,
		onSnapshot,
		query,
		orderBy,
		where,
		deleteDoc,
		doc
	} from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';
	import { getStorage, ref, getDownloadURL } from 'firebase/storage';

	let db = null;
	let storage = null;
	let homeworkList = [];

	onMount(() => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);
		db = getFirestore(app);
		storage = getStorage(app);

		// Retrieve the homework list from Firestore
		const homeworkQuery = query(
			collection(db, 'homework'),
			where('dueDate', '>=', new Date().toISOString()),
			orderBy('dueDate')
		);
		const unsubscribe = onSnapshot(homeworkQuery, (snapshot) => {
			homeworkList = snapshot.docs.map((doc) => ({
				id: doc.id,
				...doc.data()
			}));
		});

		// Clean up the listener on component unmount
		return () => {
			unsubscribe();
		};
	});

	async function getHomeworkFileURL(fileURL) {
		try {
			const storageRef = ref(storage, fileURL);
			const downloadURL = await getDownloadURL(storageRef);
			window.open(downloadURL);
		} catch (error) {
			console.error('Error retrieving homework file:', error);
		}
	}

	async function deleteHomework(homeworkId) {
		try {
			const homeworkRef = doc(db, 'homework', homeworkId);
			await deleteDoc(homeworkRef);
			console.log('Homework deleted successfully.');
		} catch (error) {
			console.error('Error deleting homework:', error);
		}
	}
</script>

<main>
	<h2>Homework List</h2>
	{#each homeworkList as homework}
		<div class="homework-item" key={homework.id}>
			<h3>{homework.homeworkName}</h3>
			<p>Task: {homework.task}</p>
			<p>Due Date: {homework.dueDate}</p>
			<p>Completed By: {homework.completedBy.join(', ')}</p>
			<p>For {homework.class} class</p>
			<button on:click={() => getHomeworkFileURL(homework.fileURL)}> Download Homework </button>
			<button on:click={() => deleteHomework(homework.id)}> Delete Homework </button>
		</div>
	{/each}
</main>
