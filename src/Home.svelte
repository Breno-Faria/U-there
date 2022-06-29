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

	let userInfo = {};
	onAuthStateChanged(auth, async (user) => {
		if (user) {
			const docRef = doc(db, "users", `${user.uid}`);
			const docSnap = await getDoc(docRef);

			if (docSnap.exists()) {
				userInfo = docSnap.data();
				console.log(docSnap.data());
				console.log(userInfo);
			}
		}
	});
</script>

{#if $isLoggedIn}
	<div class="container">
		<div class="profile">
			<h1>Bem vindo, {userInfo.username}</h1>
			<img src={userInfo.pfp} alt="" />
			<h3>{userInfo.state}</h3>
		</div>
		<div class="createGroup">
			{#if !creatingGroup}
				<button on:click={creatingGroupHandler}>Criar novo grupo</button
				>
			{/if}

			{#if creatingGroup}
				<button on:click={creatingGroupHandler}>Dismiss form</button><br
				/><br />
				<form on:submit|preventDefault={groupCreation}>
					<label for="groupTitle">Nome do grupo</label>
					<input
						bind:value={newGroup.groupName}
						type="text"
						required
						placeholder="limite de 40 caracteres"
						maxlength="40"
					/>
					<label for="groupImg">Foto do grupo (opcional)</label>
					<input
						bind:value={newGroup.groupImage}
						type="text"
						placeholder="URL da imagem"
					/>
					<button type="submit">Create group</button>
				</form>
			{/if}
		</div>
	</div>
{/if}

<style>
	.profile
	{
		margin-left: 1rem;	

	}
	.createGroup
	{
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.container {
		display: flex;
		gap: 5rem;

	}
	button {
		border-radius: 0.3rem;
		background-color: #a81d4d;
		color: white;
		font-weight: bold;
	}
</style>
