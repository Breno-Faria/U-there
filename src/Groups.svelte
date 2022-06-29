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
	const user = auth.currentUser;
	onAuthStateChanged(auth, async (user) => {
		if (user) {
			onSnapshot(doc(db, "groups", `${id}`), async (doc) => {
				users = doc.data().users;
				if (doc.data().admins.includes(user.email)) {
					isAdmin = true;
				}
				if (!doc.data().admins.includes(user.email)) {
					isAdmin = false;
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
	let addAdm = false;
	let rmAdm = false;

	const toggleAddadm = () => {
		if (!!addAdm) {
			addAdm = false;
		} else {
			addAdm = true;
			removeMember = false;
			removeGroup = false;
			rmAdm = false;
			addMember = false;
		}
	};

	const togglermAdm = () => {
		if (!!rmAdm) {
			rmAdm = false;
		} else {
			rmAdm = true;
			removeMember = false;
			removeGroup = false;
			addAdm = false;
			addMember = false;
		}
	};

	const toggleAddMember = () => {
		if (!!addMember) {
			addMember = false;
		} else {
			addMember = true;
			removeMember = false;
			removeGroup = false;
			addAdm = false;
			rmAdm = false;
		}
	};

	const toggleRemoveMember = () => {
		if (!!removeMember) {
			removeMember = false;
		} else {
			removeMember = true;
			addMember = false;
			removeGroup = false;
			addAdm = false;
			rmAdm = false;
		}
	};

	const toggleRemoveGroup = () => {
		if (!!removeGroup) {
			removeGroup = false;
		} else {
			removeGroup = true;
			addMember = false;
			removeMember = false;
			addAdm = false;
			rmAdm = false;
		}
	};

	const removeGroupHandler = async () => {
		await deleteDoc(doc(db, "groups", `${id}`));
		navigate("/");
	};

	let memberEmail = "";

	const addAdmHandler = async () => {
		const dbRef = doc(db, "groups", `${id}`);
		await updateDoc(dbRef, {
			admins: arrayUnion(`${memberEmail}`),
		});
		memberEmail = "";
		console.log("admin added")
	};

	const rmAdmHandler = async () => {
		const dbRef = doc(db, "groups", `${id}`);
		await updateDoc(dbRef, {
			admins: arrayRemove(`${memberEmail}`),
		});
		memberEmail = "";
	};

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
{#if $isLoggedIn}
	<div class="container">
		{#if isAdmin}
			<div id="adminTools">
				<h3>Ferramentas de administrador</h3>
				<div class="btnGroup">
					<button on:click={toggleAddMember}>Adicionar um novo membro</button>
					<button on:click={toggleRemoveMember}
						>Remover um membro</button
					>
					<button on:click={toggleRemoveGroup}
						>Excluir o grupo</button
					>
					<button on:click={toggleAddadm}
					>Adicionar um administrador</button>
					<button on:click={togglermAdm}
					>Remover um administrador</button>
				</div>
				{#if addMember}
					<form on:submit|preventDefault={addMemberHandler}>
						<label for="">Email</label>
						<input
							bind:value={memberEmail}
							placeholder="example@gmail.com"
							type="email"
							multiple
							required
						/>
						<button type="submit">Adicionar membro</button>
					</form>
				{/if}
				{#if removeMember}
					<form on:submit|preventDefault={removeMemberHandler}>
						<label for="">Email</label>
						<input
							bind:value={memberEmail}
							type="email"
							multiple
							required
							placeholder="example@gmail.com"
						/>
						<button type="submit">Remover membro</button>
					</form>
				{/if}
				{#if addAdm}
				<form on:submit|preventDefault={addAdmHandler}>
					<label for="">Email</label>
					<input
						bind:value={memberEmail}
						placeholder="example@gmail.com"
						type="email"
						multiple
						required
					/>
					<button type="submit">Adicionar administrador</button>
				</form>
				{/if}
				{#if rmAdm}
				<form on:submit|preventDefault={rmAdmHandler}>
					<label for="">Email</label>
					<input
						bind:value={memberEmail}
						placeholder="example@gmail.com"
						type="email"
						multiple
						required
					/>
					<button type="submit">Remover administrador</button>
				</form>
				{/if}
				{#if removeGroup}
					<button on:click={removeGroupHandler}
						>Excluir grupo (AÇÃO PERMANENTE)</button
					>
				{/if}
			</div>
		{/if}

		<div class="users">
			{#each userObjects as user}
				<div class="user">
					<img src={user.pfp} alt="Profile picture" />
					<p>{user.username}</p>
					<p id="state">{user.state}</p>
				</div>
			{/each}
		</div>
	</div>
{/if}

<style>
	#adminTools h3 {
		margin: 0;
	}
	#adminTools {
		display: flex;
		background-color: #f18db5;
		max-height: fit-content;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 1rem;
		border-radius: 20px;
		margin-top: 1%;
		gap: 5px;
	}
	button {
		border-radius: 5px;
	}

	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 3rem;
	}
	.users {
		display: flex;
		gap: 20px;
	}
	.user {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	img {
		width: 50px;
		height: 50px;
	}

	p {
		margin: 0;
	}

	#state {
		font-weight: bold;
	}
</style>
