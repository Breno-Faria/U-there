<script lang="ts">
	import { Router, Link, Route, navigate } from "svelte-navigator";
	import Home from "./Home.svelte";
	import Groups from "./Groups.svelte";
	import { db, auth } from "./db";
	import {
		FacebookAuthProvider,
		getAuth,
		GoogleAuthProvider,
		OAuthCredential,
		onAuthStateChanged,
		signInWithPopup,
		signOut,
	} from "firebase/auth";
	import {
		doc,
		collection,
		getDocs,
		onSnapshot,
		QuerySnapshot,
		query,
		where,
		addDoc,
		updateDoc,
		getDoc,
		setDoc,
	} from "firebase/firestore";
	import { writable } from "svelte/store";
	import { isLoggedIn } from "./stores";

	let groups: any = [];
	const auth = getAuth();
	onAuthStateChanged(auth, async (user) => {
		if (user) {
			onSnapshot(
				query(
					collection(db, "groups"),
					where("users", "array-contains", `${user.email}`)
				),
				(snapshot) => {
					groups = snapshot.docs.map((doc) => {
						return { ...doc.data(), id: doc.id };
					});
				},
				(error) => {
					console.log(error);
				}
			);

			$isLoggedIn = true;
			const docRef = doc(db, "users", `${user.uid}`);
			const docSnap = await getDoc(docRef);

			if (docSnap.exists()) {
				try {
					await updateDoc(docRef, {
						email: `${user.email}`,
						pfp: `${user.photoURL}`,
						username: `${user.displayName}`,
					});
				} catch (error) {
					console.log(error);
				}
			} else {
				try {
					const usersRef = collection(db, "users");
					await setDoc(doc(usersRef, `${user.uid}`), {
						email: `${user.email}`,
						pfp: `${user.photoURL}`,
						state: "online",
						username: `${user.displayName}`,
					});
				} catch (error) {
					console.log(error);
				}
			}
		}
	});

	const googleLogin = async () => {
		try {
			const provider = new GoogleAuthProvider();
			await signInWithPopup(auth, provider);
		} catch (error) {
			console.log(error);
		}
	};

	const facebookLogin = async () => {
		try {
			const provider = new FacebookAuthProvider();
			await signInWithPopup(auth, provider);
		} catch (error) {
			console.log(error);
		}
	};

	const signOutHandler = async () => {
		signOut(auth)
			.then(() => {
				$isLoggedIn = false;
			})
			.catch((error) => {
				console.log(error);
			});
		navigate("/");
	};

	const disponivelChangeState = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		const userInfo = doc(db, "users", `${user.uid}`);
		await updateDoc(userInfo, {
			state: "🟢 Disponivel",
		});
	};
	const ocupadoChangeState = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		const userInfo = doc(db, "users", `${user.uid}`);
		await updateDoc(userInfo, {
			state: "🟡 Ocupado",
		});
	};
	const offlineChangeState = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		const userInfo = doc(db, "users", `${user.uid}`);
		await updateDoc(userInfo, {
			state: "🔴 Offline",
		});
	};
	const feriasChangeState = async () => {
		const auth = getAuth();
		const user = auth.currentUser;
		const userInfo = doc(db, "users", `${user.uid}`);
		await updateDoc(userInfo, {
			state: "De férias 🌴🥳🎉",
		});
	};

	
	const user = auth.currentUser;
</script>

<body>
	<main>
		<Router>
			<nav class="header">
				<img alt="Logo" src="/icon.png" />
				{#if !$isLoggedIn}
					<button on:click={googleLogin}>Log-in com Google</button>
					<button on:click={facebookLogin}
						>Log-in com Facebook</button
					>
				{/if}

				{#if $isLoggedIn}

					<div class="options">
						<button on:click={signOutHandler}>Sign out</button>
						<Link to="/"><button id="home">Home</button></Link>
					</div>
					<div class="states">
						<button on:click={disponivelChangeState}
							>Disponível</button
						>
						<button on:click={ocupadoChangeState}>Ocupado</button>
						<button on:click={offlineChangeState}>Offline</button>
						<button on:click={feriasChangeState}
							>De férias 🌴🥳🎉</button
						>
					</div>
					{#each groups as group}
						{#if group.groupImage != ""}
							<a href="/groups/{group.id}">
								<div class="group">
									<img
										class="pfp"
										src={group.groupImage}
										alt="group icon"
									/>
									<p>{group.groupName}</p>
								</div>
							</a>
						{:else}
							<a href="/groups/{group.id}">
								<div class="group">
									<img
										class="pfp"
										src="/defaultgroup.png"
										alt="group icon"
									/>
									<p>{group.groupName}</p>
								</div>
							</a>
						{/if}
					{/each}
				{/if}
			</nav>
			<Route path="groups/:id" component={Groups} />
			<Route path="/" component={Home} />
		</Router>
	</main>
</body>

<slot />

<style>
	.group p {
		margin: 0;
	}

	.group {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		font-size: x-large;
	}
	:global(a) {
		text-decoration: none;
		color: black;
	}
	#home {
		width: 5rem;
	}
	button {
		border-radius: 0.3rem;
		background-color: #a81d4d;
		color: white;
		font-weight: bold;
	}
	.states {
		display: flex;
		flex-direction: column;
		gap: 0.2rem;
	}
	.options {
		gap: 1rem;
		display: flex;
		flex-direction: column;
	}
	.header {
		display: flex;
		gap: 2rem;
		justify-content: flex-start;
		align-items: center;
		padding: 1rem;
	}
	.pfp {
		width: 100px;
		height: auto;
		max-height: 150px;
	}
	:global(body) {
		margin: 0;
		background-color: #94dee2;
	}
	img {
		width: 150px;
		height: auto;
	}

	nav {
		display: flex;
		background-color: #f18db5;
		overflow-y:hidden;
	}
</style>
