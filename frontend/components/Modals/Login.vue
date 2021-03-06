<template>
	<div class="modal is-active">
		<div
			class="modal-background"
			@click="
				closeModal({
					sector: 'header',
					modal: 'login'
				})
			"
		/>
		<div class="modal-card">
			<header class="modal-card-head">
				<p class="modal-card-title">
					Login
				</p>
				<button
					class="delete"
					@click="
						closeModal({
							sector: 'header',
							modal: 'login'
						})
					"
				/>
			</header>
			<section class="modal-card-body">
				<!-- validation to check if exists http://bulma.io/documentation/elements/form/ -->
				<form>
					<label class="label">Email</label>
					<p class="control">
						<input
							v-model="email"
							class="input"
							type="email"
							placeholder="Email..."
						/>
					</p>
					<label class="label">Password</label>
					<p class="control">
						<input
							v-model="password"
							class="input"
							type="password"
							placeholder="Password..."
							@keypress="
								$parent.submitOnEnter(submitModal, $event)
							"
						/>
					</p>
					<p>
						By logging in/registering you agree to our
						<router-link to="/terms"> Terms of Service </router-link
						>&nbsp;and
						<router-link to="/privacy"> Privacy Policy </router-link
						>.
					</p>
				</form>
			</section>
			<footer class="modal-card-foot">
				<a class="button is-primary" href="#" @click="submitModal()"
					>Submit</a
				>
				<a
					class="button is-github"
					:href="serverDomain + '/auth/github/authorize'"
					@click="githubRedirect()"
				>
					<div class="icon">
						<img class="invert" src="/assets/social/github.svg" />
					</div>
					&nbsp;&nbsp;Login with GitHub
				</a>
				<a href="/reset_password" v-on:click="resetPassword()"
					>Forgot password?</a
				>
			</footer>
		</div>
	</div>
</template>

<script>
import { mapActions } from "vuex";

import Toast from "toasters";

export default {
	data() {
		return {
			email: "",
			password: "",
			serverDomain: ""
		};
	},
	methods: {
		submitModal() {
			this.login({
				email: this.email,
				password: this.password
			})
				.then(res => {
					if (res.status === "success") window.location.reload();
				})
				.catch(
					err => new Toast({ content: err.message, timeout: 5000 })
				);
		},
		resetPassword() {
			this.closeModal({ sector: "header", modal: "login" });
			this.$router.go("/reset_password");
		},
		githubRedirect() {
			localStorage.setItem("github_redirect", this.$route.path);
		},
		...mapActions("modals", ["closeModal"]),
		...mapActions("user/auth", ["login"])
	},
	mounted() {
		lofig.get("serverDomain").then(serverDomain => {
			this.serverDomain = serverDomain;
		});
	}
};
</script>

<style lang="scss" scoped>
@import "scss/variables/colors.scss";

.button.is-github {
	background-color: $dark-grey-2;
	color: $white !important;
}

.is-github:focus {
	background-color: $dark-grey-3;
}
.is-primary:focus {
	background-color: $primary-color !important;
}

.invert {
	filter: brightness(5);
}

a {
	color: $primary-color;
}
</style>
