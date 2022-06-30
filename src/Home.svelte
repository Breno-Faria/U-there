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

	let userInfo = {
		username: "",
		pfp: "",
		state: "",
	};
	onAuthStateChanged(auth, async (user) => {
		if (user) {
			const docRef = doc(db, "users", `${user.uid}`);
			onSnapshot(docRef, (snapshot) => {
				userInfo.pfp = snapshot.data().pfp;
				userInfo.username = snapshot.data().username;
				userInfo.state = snapshot.data().state;
			});
		}
	});

	let number = "";
	const addPhoneNumber = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		const userInfo = doc(db, "users", `${user.uid}`);
		await updateDoc(userInfo, {
			number: `${number}`,
		});
		number = "";
	};
</script>

{#if $isLoggedIn}
	<div class="container">
		<div class="profile">
			<h1>Bem-vindo, {userInfo.username}</h1>
			<img src={userInfo.pfp} alt="" style="border-radius: 10px" />
			<h3>{userInfo.state}</h3>
			<br />
			<form on:submit|preventDefault={addPhoneNumber}>
				<h4>Adicione/atualize seu número de telefone!</h4>
				<p>
					Para que outros membros do grupo possam iniciar conversas no
					Whatsapp, é preciso adicionar o seu numero de telefone.
					Remova todos os s ímbolos, e insira apenas os algarismos,
					includindo o DDD no formato: 21888888888
				</p>
				<input bind:value={number} type="number" /> <button type="submit">Confirmar</button>
			</form>
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
	input[type="number"]::-webkit-inner-spin-button,
	input[type="number"]::-webkit-outer-spin-button {
		-webkit-appearance: none;
		-moz-appearance: none;
		appearance: none;
	}
	.profile {
		max-width: 30vw;
	}

	.createGroup {
		display: flex;
		align-items: flex-start;
		justify-content: center;
		gap: 1rem;
	}
	.container {
		display: flex;
		gap: 5rem;
		margin: 0 1rem;
		flex-wrap: wrap;
	}
	button {
		border-radius: 0.3rem;
		background-color: #a81d4d;
		color: white;
		font-weight: bold;
	}
</style>
