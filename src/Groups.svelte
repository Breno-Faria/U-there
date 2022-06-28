<script lang="ts">
	import { getAuth, onAuthStateChanged } from "firebase/auth";
	import {
		arrayRemove,
		arrayUnion,
		collection,
		deleteDoc,
		doc,
		getDoc,
		getDocs,
		onSnapshot,
		query,
		setDoc,
		updateDoc,
		where,
	} from "firebase/firestore";
	import { navigate } from "svelte-navigator";
	import { db, auth } from "./db";
	import { isLoggedIn } from "./stores";
	export let id: string;
	let users: any = [];
	let userObjects: any = [];
	let isAdmin = false;
	const auth = getAuth();
	onAuthStateChanged(auth, async (user) => {
		if (user) {
			onSnapshot(doc(db, "groups", `${id}`), async (doc) => {
				users = doc.data().users;
				if (doc.data().admins.includes(user.email)) {
					isAdmin = true;
				}
				if (!users.includes(user.email) || !$isLoggedIn) {
					navigate("/");
				}

				onSnapshot(
					query(collection(db, "users"), where("email", "in", users)),
					(snapshot) => {
						userObjects = snapshot.docs.map((doc) => {
							return { ...doc.data(), id: doc.id };
						});
					},
					(error) => {
						console.log(error);
					}
				);
			});
		}
	});
	let addMember = false;
	let removeMember = false;
	let removeGroup = false;

	const toggleAddMember = () => {
		if (!!addMember) {
			addMember = false;
		} else {
			addMember = true;
			removeMember = false;
			removeGroup = false;
		}
	};

	const toggleRemoveMember = () => {
		if (!!removeMember) {
			removeMember = false;
		} else {
			removeMember = true;
			addMember = false;
			removeGroup = false;
		}
	};

	const toggleRemoveGroup = () => {
		if (!!removeGroup) {
			removeGroup = false;
		} else {
			removeGroup = true;
			addMember = false;
			removeMember = false;
		}
	};

	const removeGroupHandler = async () => {
		await deleteDoc(doc(db, "groups", `${id}`));
		navigate("/");
	};

	let memberEmail = "";
	const addMemberHandler = async () => {
		const dbRef = doc(db, "groups", `${id}`);
		await updateDoc(dbRef, {
			users: arrayUnion(`${memberEmail}`),
		});
		memberEmail = "";
	};

	const removeMemberHandler = async () => {
		const dbRef = doc(db, "groups", `${id}`);
		await updateDoc(dbRef, {
			users: arrayRemove(`${memberEmail}`),
		});
		memberEmail = "";
	};
</script>

{#if isAdmin}
	<button on:click={toggleAddMember}>Add a new member</button>
	<button on:click={toggleRemoveMember}>Remove a member</button>
	<button on:click={toggleRemoveGroup}>Delete the group</button>
	{#if addMember}
		<form on:submit|preventDefault={addMemberHandler}>
			<label for="">New member's email</label>
			<input
				bind:value={memberEmail}
				placeholder="example@gmail.com"
				type="email"
				multiple
				required
			/>
			<button type="submit">Add member</button>
		</form>
	{/if}
	{#if removeMember}
		<form on:submit|preventDefault={removeMemberHandler}>
			<label for="">Member's email</label>
			<input
				bind:value={memberEmail}
				type="email"
				multiple
				required
				placeholder="example@gmail.com"
			/>
			<button type="submit">Remove member</button>
		</form>
	{/if}
	{#if removeGroup}
		<br />
		<button on:click={removeGroupHandler}
			>Delete group (PERMANENT ACTION)</button
		>
	{/if}
{/if}

<div class="users">
	{#each userObjects as user}
		<div class="user">
			<img src={user.pfp} alt="Profile picture" />
			{user.username} <br>
			{user.state}
		</div>
	{/each}
</div>

<style>
	.users
	{
		display: flex;
		gap: 20px;
	}
	.user {
		display: flex;
		flex-direction: column;
	}

	img {
		width: 50px;
		height: 50px;
	}
</style>
