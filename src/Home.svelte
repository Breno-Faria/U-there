<script lang="ts">
	import { db, auth } from "./db";
	import {
		addDoc,
		collection,
		doc,
		getDoc,
		onSnapshot,
		query,
		updateDoc,
		where,
	} from "firebase/firestore";
	import { getAuth, onAuthStateChanged } from "firebase/auth";
	import { isLoggedIn } from "./stores";

	let newGroup = {
		groupName: "",
		users: [],
		groupImage: "",
		admins: [],
	};

	const groupCreation = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		newGroup.users.push(user.email);
		newGroup.admins.push(user.email);

		await addDoc(collection(db, "groups"), newGroup);

		newGroup.groupName = "";
		newGroup.users = [];
		newGroup.groupImage = "";
		newGroup.admins = [];
	};

	let creatingGroup: boolean = false;
	const creatingGroupHandler = () => {
		if (creatingGroup) {
			creatingGroup = false;
		} else {
			creatingGroup = true;
		}
	};
</script>

{#if $isLoggedIn}
	{#if !creatingGroup}
		<button on:click={creatingGroupHandler}>Criar novo grupo</button>
	{/if}

	{#if creatingGroup}
		<button on:click={creatingGroupHandler}>Dismiss form</button><br /><br
		/>
		<form on:submit|preventDefault={groupCreation}>
			<label for="groupTitle">Group title</label>
			<input
				bind:value={newGroup.groupName}
				type="text"
				required
				placeholder="Title"
			/>
			<label for="groupImg">Group image (URL)</label>
			<input
				bind:value={newGroup.groupImage}
				type="text"
				placeholder="Insert an image URL here"
			/>
			<button type="submit">Create group</button>
		</form>
	{/if}
{/if}

<style>
</style>
