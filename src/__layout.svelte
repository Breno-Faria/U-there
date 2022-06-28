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
	import { writable } from 'svelte/store';
	import { isLoggedIn } from './stores';
	
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
	};

</script>

<body style="margin: 0;">
	<main>
		<Router>
			<nav class="header">
				<img alt="Logo" src="/icon.png" />
				{#if !$isLoggedIn}
					<button on:click={googleLogin}>Log in with Google</button>
					<button on:click={facebookLogin}
						>Log in with Facebook</button
					>
				{/if}

				{#if $isLoggedIn}
					<button on:click={signOutHandler}>Sign out</button>
					<Link to="/">Home</Link>
					{#each groups as group}
						<div class="group">
							<a href="/groups/{group.id}">
								<img
									class="pfp"
									src={group.groupImage}
									alt="group icon"
								/>
								<p>{group.groupName}</p>
							</a>
						</div>
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
	.header
	{
		display: flex;
		gap: 30px;
		justify-content: flex-start;
		align-items: center;
	}
	.pfp {
		width: 50px;
		height: 50px;
	}
	:global(body) {
		margin: 0;
	}
	img {
		width: 100px;
		height: auto;
	}

	nav {
		background-color: pink;
	}
	button {
		padding: 0;
		background: none;
		border: none;
	}
</style>
