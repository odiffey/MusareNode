<template>
	<modal title="Edit Station" class="edit-station-modal">
		<template v-slot:body>
			<div class="section left-section">
				<div class="col col-2">
					<div>
						<label class="label">Name</label>
						<p class="control">
							<input
								class="input"
								type="text"
								v-model="editing.name"
							/>
						</p>
					</div>
					<div>
						<label class="label">Display name</label>
						<p class="control">
							<input
								class="input"
								type="text"
								v-model="editing.displayName"
							/>
						</p>
					</div>
				</div>
				<div class="col col-1">
					<div>
						<label class="label">Description</label>
						<p class="control">
							<input
								class="input"
								type="text"
								v-model="editing.description"
							/>
						</p>
					</div>
				</div>
				<div class="col col-2" v-if="editing.genres">
					<div>
						<label class="label">Genre(s)</label>
						<p class="control has-addons">
							<input
								class="input"
								type="text"
								id="new-genre"
								v-model="genreInputValue"
								v-on:blur="blurGenreInput()"
								v-on:focus="focusGenreInput()"
								v-on:keydown="keydownGenreInput()"
								v-on:keyup.enter="addTag('genres')"
							/>
							<button
								class="button is-info add-button blue"
								v-on:click="addTag('genres')"
							>
								<i class="material-icons">add</i>
							</button>
						</p>
						<div
							class="autosuggest-container"
							v-if="
								(genreInputFocussed ||
									genreAutosuggestContainerFocussed) &&
									genreAutosuggestItems.length > 0
							"
							@mouseover="focusGenreContainer()"
							@mouseleave="blurGenreContainer()"
						>
							<span
								class="autosuggest-item"
								tabindex="0"
								v-on:click="selectGenreAutosuggest(item)"
								v-for="(item, index) in genreAutosuggestItems"
								:key="index"
								>{{ item }}</span
							>
						</div>
						<div class="list-container">
							<div
								class="list-item"
								v-for="(genre, index) in editing.genres"
								:key="index"
							>
								<div
									class="list-item-circle blue"
									v-on:click="removeTag('genres', index)"
								>
									<i class="material-icons">close</i>
								</div>
								<p>{{ genre }}</p>
							</div>
						</div>
					</div>
					<div>
						<label class="label">Blacklist genre(s)</label>
						<p class="control has-addons">
							<input
								class="input"
								type="text"
								v-model="blacklistGenreInputValue"
								v-on:blur="blurBlacklistGenreInput()"
								v-on:focus="focusBlacklistGenreInput()"
								v-on:keydown="keydownBlacklistGenreInput()"
								v-on:keyup.enter="addTag('blacklist-genres')"
							/>
							<button
								class="button is-info add-button red"
								v-on:click="addTag('blacklist-genres')"
							>
								<i class="material-icons">add</i>
							</button>
						</p>
						<div
							class="autosuggest-container"
							v-if="
								(blacklistGenreInputFocussed ||
									blacklistGenreAutosuggestContainerFocussed) &&
									blacklistGenreAutosuggestItems.length > 0
							"
							@mouseover="focusBlacklistGenreContainer()"
							@mouseleave="blurBlacklistGenreContainer()"
						>
							<span
								class="autosuggest-item"
								tabindex="0"
								v-on:click="
									selectBlacklistGenreAutosuggest(item)
								"
								v-for="(item,
								index) in blacklistGenreAutosuggestItems"
								:key="index"
								>{{ item }}</span
							>
						</div>
						<div class="list-container">
							<div
								class="list-item"
								v-for="(genre,
								index) in editing.blacklistedGenres"
								:key="index"
							>
								<div
									class="list-item-circle red"
									v-on:click="
										removeTag('blacklist-genres', index)
									"
								>
									<i class="material-icons">close</i>
								</div>
								<p>{{ genre }}</p>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="section right-section">
				<div>
					<label class="label">Privacy</label>
					<div
						@mouseenter="privacyDropdownActive = true"
						@mouseleave="privacyDropdownActive = false"
						class="button-wrapper"
					>
						<button
							v-bind:class="{
								green: true,
								current: editing.privacy === 'public'
							}"
							v-if="
								privacyDropdownActive ||
									editing.privacy === 'public'
							"
							@click="updatePrivacyLocal('public')"
						>
							<i class="material-icons">people</i>
							Public
						</button>
						<button
							v-bind:class="{
								orange: true,
								current: editing.privacy === 'unlisted'
							}"
							v-if="
								privacyDropdownActive ||
									editing.privacy === 'unlisted'
							"
							@click="updatePrivacyLocal('unlisted')"
						>
							<i class="material-icons">link</i>
							Unlisted
						</button>
						<button
							v-bind:class="{
								red: true,
								current: editing.privacy === 'private'
							}"
							v-if="
								privacyDropdownActive ||
									editing.privacy === 'private'
							"
							@click="updatePrivacyLocal('private')"
						>
							<i class="material-icons">lock</i>
							Private
						</button>
					</div>
				</div>
				<div v-if="editing.type === 'community'">
					<label class="label">Mode</label>
					<div
						@mouseenter="modeDropdownActive = true"
						@mouseleave="modeDropdownActive = false"
						class="button-wrapper"
					>
						<button
							v-bind:class="{
								blue: true,
								current: editing.partyMode === false
							}"
							v-if="modeDropdownActive || !editing.partyMode"
							@click="updatePartyModeLocal(false)"
						>
							<i class="material-icons">queue_music</i>
							Playlist
						</button>
						<button
							v-bind:class="{
								yellow: true,
								current: editing.partyMode === true
							}"
							v-if="
								modeDropdownActive || editing.partyMode === true
							"
							@click="updatePartyModeLocal(true)"
						>
							<i class="material-icons">post_add</i>
							Party
						</button>
					</div>
				</div>
				<div
					v-if="
						editing.type === 'community' &&
							editing.partyMode === true
					"
				>
					<label class="label">Queue lock</label>
					<div
						@mouseenter="queueLockDropdownActive = true"
						@mouseleave="queueLockDropdownActive = false"
						class="button-wrapper"
					>
						<button
							v-bind:class="{
								green: true,
								current: editing.locked
							}"
							v-if="queueLockDropdownActive || editing.locked"
							@click="updateQueueLockLocal(true)"
						>
							<i class="material-icons">done</i>
							On
						</button>
						<button
							v-bind:class="{
								red: true,
								current: !editing.locked
							}"
							v-if="queueLockDropdownActive || !editing.locked"
							@click="updateQueueLockLocal(false)"
						>
							<i class="material-icons">not_interested</i>
							Off
						</button>
					</div>
				</div>
			</div>
		</template>
		<template v-slot:footer>
			<button class="button is-success" v-on:click="update()">
				Update Settings
			</button>
			<button
				v-if="station.type === 'community'"
				class="button is-danger"
				@click="deleteStation()"
			>
				Delete station
			</button>
		</template>
	</modal>
</template>

<script>
import { mapState, mapActions } from "vuex";

import Toast from "toasters";
import Modal from "./Modal.vue";
import io from "../../io";
import validation from "../../validation";

export default {
	computed: mapState({
		editing(state) {
			return this.$props.store.split("/").reduce((a, v) => a[v], state)
				.editing;
		},
		station(state) {
			return this.$props.store.split("/").reduce((a, v) => a[v], state)
				.station;
		}
	}),
	mounted() {
		io.getSocket(socket => {
			this.socket = socket;
			return socket;
		});
	},
	data() {
		return {
			genreInputValue: "",
			genreInputFocussed: false,
			genreAutosuggestContainerFocussed: false,
			keydownGenreInputTimeout: 0,
			genreAutosuggestItems: [],
			blacklistGenreInputValue: "",
			blacklistGenreInputFocussed: false,
			blacklistGenreAutosuggestContainerFocussed: false,
			blacklistKeydownGenreInputTimeout: 0,
			blacklistGenreAutosuggestItems: [],
			privacyDropdownActive: false,
			modeDropdownActive: false,
			queueLockDropdownActive: false,
			genres: [
				"Blues",
				"Country",
				"Disco",
				"Funk",
				"Hip-Hop",
				"Jazz",
				"Metal",
				"Oldies",
				"Other",
				"Pop",
				"Rap",
				"Reggae",
				"Rock",
				"Techno",
				"Trance",
				"Classical",
				"Instrumental",
				"House",
				"Electronic",
				"Christian Rap",
				"Lo-Fi",
				"Musical",
				"Rock 'n' Roll",
				"Opera",
				"Drum & Bass",
				"Club-House",
				"Indie",
				"Heavy Metal",
				"Christian rock",
				"Dubstep"
			]
		};
	},
	props: ["store"],
	methods: {
		update() {
			if (this.station.name !== this.editing.name) this.updateName();
			if (this.station.displayName !== this.editing.displayName)
				this.updateDisplayName();
			if (this.station.description !== this.editing.description)
				this.updateDescription();
			if (this.station.privacy !== this.editing.privacy)
				this.updatePrivacy();
			if (
				this.station.type === "community" &&
				this.station.partyMode !== this.editing.partyMode
			)
				this.updatePartyMode();
			if (
				this.station.type === "community" &&
				this.editing.partyMode &&
				this.station.locked !== this.editing.locked
			)
				this.updateQueueLock();
			if (this.$props.store !== "station") {
				if (
					this.station.genres.toString() !==
					this.editing.genres.toString()
				)
					this.updateGenres();
				if (
					this.station.blacklistedGenres.toString() !==
					this.editing.blacklistedGenres.toString()
				)
					this.updateBlacklistedGenres();
			}
		},
		updateName() {
			const { name } = this.editing;
			if (!validation.isLength(name, 2, 16))
				return new Toast({
					content: "Name must have between 2 and 16 characters.",
					timeout: 8000
				});
			if (!validation.regex.az09_.test(name))
				return new Toast({
					content:
						"Invalid name format. Allowed characters: a-z, 0-9 and _.",
					timeout: 8000
				});

			return this.socket.emit(
				"stations.updateName",
				this.editing._id,
				name,
				res => {
					if (res.status === "success") {
						if (this.station) this.station.name = name;
						else {
							this.$parent.stations.forEach((station, index) => {
								if (station._id === this.editing._id) {
									this.$parent.stations[index].name = name;
									return name;
								}

								return false;
							});
						}
					}

					new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updateDisplayName() {
			const { displayName } = this.editing;
			if (!validation.isLength(displayName, 2, 32))
				return new Toast({
					content:
						"Display name must have between 2 and 32 characters.",
					timeout: 8000
				});
			if (!validation.regex.ascii.test(displayName))
				return new Toast({
					content:
						"Invalid display name format. Only ASCII characters are allowed.",
					timeout: 8000
				});

			return this.socket.emit(
				"stations.updateDisplayName",
				this.editing._id,
				displayName,
				res => {
					if (res.status === "success") {
						if (this.station)
							this.station.displayName = displayName;
						else {
							this.$parent.stations.forEach((station, index) => {
								if (station._id === this.editing._id) {
									this.$parent.stations[
										index
									].displayName = displayName;
									return displayName;
								}

								return false;
							});
						}
					}

					new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updateDescription() {
			const { description } = this.editing;
			if (!validation.isLength(description, 2, 200))
				return new Toast({
					content:
						"Description must have between 2 and 200 characters.",
					timeout: 8000
				});

			let characters = description.split("");
			characters = characters.filter(character => {
				return character.charCodeAt(0) === 21328;
			});
			if (characters.length !== 0)
				return new Toast({
					content: "Invalid description format.",
					timeout: 8000
				});

			return this.socket.emit(
				"stations.updateDescription",
				this.editing._id,
				description,
				res => {
					if (res.status === "success") {
						if (this.station)
							this.station.description = description;
						else {
							this.$parent.stations.forEach((station, index) => {
								if (station._id === this.editing._id) {
									this.$parent.stations[
										index
									].description = description;
									return description;
								}

								return false;
							});
						}

						return new Toast({
							content: res.message,
							timeout: 4000
						});
					}

					return new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updatePrivacyLocal(privacy) {
			if (this.editing.privacy === privacy) return;
			this.editing.privacy = privacy;
			this.privacyDropdownActive = false;
		},
		updatePrivacy() {
			this.socket.emit(
				"stations.updatePrivacy",
				this.editing._id,
				this.editing.privacy,
				res => {
					if (res.status === "success") {
						if (this.station)
							this.station.privacy = this.editing.privacy;
						else {
							this.$parent.stations.forEach((station, index) => {
								if (station._id === this.editing._id) {
									this.$parent.stations[
										index
									].privacy = this.editing.privacy;
									return this.editing.privacy;
								}

								return false;
							});
						}
						return new Toast({
							content: res.message,
							timeout: 4000
						});
					}

					return new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updateGenres() {
			this.socket.emit(
				"stations.updateGenres",
				this.editing._id,
				this.editing.genres,
				res => {
					if (res.status === "success") {
						const genres = JSON.parse(
							JSON.stringify(this.editing.genres)
						);
						if (this.station) this.station.genres = genres;
						this.$parent.stations.forEach((station, index) => {
							if (station._id === this.editing._id) {
								this.$parent.stations[index].genres = genres;
								return genres;
							}

							return false;
						});

						return new Toast({
							content: res.message,
							timeout: 4000
						});
					}

					return new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updateBlacklistedGenres() {
			this.socket.emit(
				"stations.updateBlacklistedGenres",
				this.editing._id,
				this.editing.blacklistedGenres,
				res => {
					if (res.status === "success") {
						const blacklistedGenres = JSON.parse(
							JSON.stringify(this.editing.blacklistedGenres)
						);
						if (this.station)
							this.station.blacklistedGenres = blacklistedGenres;
						this.$parent.stations.forEach((station, index) => {
							if (station._id === this.editing._id) {
								this.$parent.stations[
									index
								].blacklistedGenres = blacklistedGenres;
								return blacklistedGenres;
							}

							return false;
						});
						return new Toast({
							content: res.message,
							timeout: 4000
						});
					}

					return new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updatePartyModeLocal(partyMode) {
			if (this.editing.partyMode === partyMode) return;
			this.editing.partyMode = partyMode;
			this.modeDropdownActive = false;
		},
		updatePartyMode() {
			this.socket.emit(
				"stations.updatePartyMode",
				this.editing._id,
				this.editing.partyMode,
				res => {
					if (res.status === "success") {
						if (this.station)
							this.station.partyMode = this.editing.partyMode;
						// if (this.station)
						// 	this.station.partyMode = this.editing.partyMode;
						// this.$parent.stations.forEach((station, index) => {
						// 	if (station._id === this.editing._id) {
						// 		this.$parent.stations[
						// 			index
						// 		].partyMode = this.editing.partyMode;
						// 		return this.editing.partyMode;
						// 	}

						// 	return false;
						// });

						return new Toast({
							content: res.message,
							timeout: 4000
						});
					}

					return new Toast({ content: res.message, timeout: 8000 });
				}
			);
		},
		updateQueueLockLocal(locked) {
			if (this.editing.locked === locked) return;
			this.editing.locked = locked;
			this.queueLockDropdownActive = false;
		},
		updateQueueLock() {
			this.socket.emit("stations.toggleLock", this.editing._id, res => {
				console.log(res);
				if (res.status === "success") {
					if (this.station) this.station.locked = res.data;
					return new Toast({
						content: `Toggled queue lock succesfully to ${res.data}`,
						timeout: 4000
					});
				}
				return new Toast({
					content: "Failed to toggle queue lock.",
					timeout: 8000
				});
			});
		},
		deleteStation() {
			this.socket.emit("stations.remove", this.editing._id, res => {
				if (res.status === "success")
					this.closeModal({
						sector: "station",
						modal: "editStation"
					});
				return new Toast({ content: res.message, timeout: 8000 });
			});
		},
		blurGenreInput() {
			this.genreInputFocussed = false;
		},
		focusGenreInput() {
			this.genreInputFocussed = true;
		},
		keydownGenreInput() {
			clearTimeout(this.keydownGenreInputTimeout);
			this.keydownGenreInputTimeout = setTimeout(() => {
				if (this.genreInputValue.length > 1) {
					this.genreAutosuggestItems = this.genres.filter(genre => {
						return genre
							.toLowerCase()
							.startsWith(this.genreInputValue.toLowerCase());
					});
				} else this.genreAutosuggestItems = [];
			}, 1000);
		},
		focusGenreContainer() {
			this.genreAutosuggestContainerFocussed = true;
		},
		blurGenreContainer() {
			this.genreAutosuggestContainerFocussed = false;
		},
		selectGenreAutosuggest(value) {
			this.genreInputValue = value;
		},
		blurBlacklistGenreInput() {
			this.blacklistGenreInputFocussed = false;
		},
		focusBlacklistGenreInput() {
			this.blacklistGenreInputFocussed = true;
		},
		keydownBlacklistGenreInput() {
			clearTimeout(this.keydownBlacklistGenreInputTimeout);
			this.keydownBlacklistGenreInputTimeout = setTimeout(() => {
				console.log(123, this.blacklistGenreInputValue);
				if (this.blacklistGenreInputValue.length > 1) {
					console.log(333);
					this.blacklistGenreAutosuggestItems = this.genres.filter(
						genre => {
							console.log(444);
							return genre
								.toLowerCase()
								.startsWith(
									this.blacklistGenreInputValue.toLowerCase()
								);
						}
					);
				} else this.blacklistGenreAutosuggestItems = [];
			}, 1000);
		},
		focusBlacklistGenreContainer() {
			this.blacklistGenreAutosuggestContainerFocussed = true;
		},
		blurBlacklistGenreContainer() {
			this.blacklistGenreAutosuggestContainerFocussed = false;
		},
		selectBlacklistGenreAutosuggest(value) {
			this.blacklistGenreInputValue = value;
		},
		addTag(type) {
			if (type === "genres") {
				const genre = this.genreInputValue.toLowerCase().trim();
				if (this.editing.genres.indexOf(genre) !== -1)
					return new Toast({
						content: "Genre already exists",
						timeout: 3000
					});
				if (genre) {
					this.editing.genres.push(genre);
					this.genreInputValue = "";
					return false;
				}

				return new Toast({
					content: "Genre cannot be empty",
					timeout: 3000
				});
			}
			if (type === "blacklist-genres") {
				const genre = this.blacklistGenreInputValue
					.toLowerCase()
					.trim();
				if (this.editing.blacklistedGenres.indexOf(genre) !== -1)
					return new Toast({
						content: "Blacklist genre already exists",
						timeout: 3000
					});
				if (genre) {
					this.editing.blacklistedGenres.push(genre);
					this.blacklistGenreInputValue = "";
					return false;
				}

				return new Toast({
					content: "Blacklist genre cannot be empty",
					timeout: 3000
				});
			}

			return false;
		},
		removeTag(type, index) {
			if (type === "genres") this.editing.genres.splice(index, 1);
			else if (type === "blacklist-genres")
				this.editing.blacklistedGenres.splice(index, 1);
		},
		...mapActions("modals", ["closeModal"])
	},
	components: { Modal }
};
</script>

<style lang="scss">
.edit-station-modal {
	.modal-card-title {
		text-align: center;
		margin-left: 24px;
	}

	.modal-card {
		width: 800px;
		height: 550px;

		.modal-card-body {
			padding: 16px;
			display: flex;
		}
	}
}
</style>

<style lang="scss" scoped>
@import "scss/variables/colors.scss";

.section {
	border: 1px solid #a3e0ff;
	background-color: #f4f4f4;
	border-radius: 5px;
	padding: 16px;
}

.left-section {
	width: 595px;
	display: grid;
	gap: 16px;
	grid-template-rows: min-content min-content auto;

	.control {
		input {
			width: 100%;
		}

		.add-button {
			width: 32px;

			&.blue {
				background-color: $musareBlue !important;
			}

			&.red {
				background-color: $red !important;
			}

			i {
				font-size: 32px;
			}
		}
	}

	.col {
		> div {
			position: relative;
		}
	}

	.list-item-circle {
		width: 16px;
		height: 16px;
		border-radius: 8px;
		cursor: pointer;
		margin-right: 8px;
		float: left;
		-webkit-touch-callout: none;
		-webkit-user-select: none;
		-khtml-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
		user-select: none;

		&.blue {
			background-color: $musareBlue;

			i {
				color: $musareBlue;
			}
		}

		&.red {
			background-color: $red;

			i {
				color: $red;
			}
		}

		i {
			font-size: 14px;
			margin-left: 1px;
		}
	}

	.list-item-circle:hover,
	.list-item-circle:focus {
		i {
			color: white;
		}
	}

	.list-item > p {
		line-height: 16px;
		word-wrap: break-word;
		width: calc(100% - 24px);
		left: 24px;
		float: left;
		margin-bottom: 8px;
	}

	.list-item:last-child > p {
		margin-bottom: 0;
	}

	.autosuggest-container {
		position: absolute;
		background: white;
		width: calc(100% + 1px);
		top: 57px;
		z-index: 200;
		overflow: auto;
		max-height: 100%;
		clear: both;

		.autosuggest-item {
			padding: 8px;
			display: block;
			border: 1px solid #dbdbdb;
			margin-top: -1px;
			line-height: 16px;
			cursor: pointer;
			-webkit-user-select: none;
			-ms-user-select: none;
			-moz-user-select: none;
			user-select: none;
		}

		.autosuggest-item:hover,
		.autosuggest-item:focus {
			background-color: #eee;
		}

		.autosuggest-item:first-child {
			border-top: none;
		}

		.autosuggest-item:last-child {
			border-radius: 0 0 3px 3px;
		}
	}
}

.right-section {
	width: 157px;
	margin-left: 16px;
	display: grid;
	gap: 16px;
	grid-template-rows: min-content min-content min-content;

	.button-wrapper {
		display: flex;
		flex-direction: column;
	}

	button {
		width: 100%;
		height: 36px;
		border: 0;
		border-radius: 10px;
		font-size: 18px;
		color: white;
		box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.25);
		display: block;
		text-align: center;
		justify-content: center;
		display: inline-flex;
		-ms-flex-align: center;
		align-items: center;
		-moz-user-select: none;
		user-select: none;
		cursor: pointer;
		margin-bottom: 16px;
		padding: 0;

		&.current {
			order: -1;
		}

		&.red {
			&.current,
			&:hover {
				background-color: $red;
			}
			background-color: rgba($red, 0.7);
			transition: all 0.3s ease-in-out;
		}

		&.green {
			&.current,
			&:hover {
				background-color: $green;
			}
			background-color: rgba($green, 0.7);
			transition: all 0.3s ease-in-out;
		}

		&.blue {
			&.current,
			&:hover {
				background-color: $musareBlue;
			}
			background-color: rgba($musareBlue, 0.7);
			transition: all 0.3s ease-in-out;
		}

		&.orange {
			&.current,
			&:hover {
				background-color: $light-orange;
			}
			background-color: rgba($light-orange, 0.7);
			transition: all 0.3s ease-in-out;
		}

		&.yellow {
			&.current,
			&:hover {
				background-color: $yellow;
			}
			background-color: rgba($yellow, 0.7);
			transition: all 0.3s ease-in-out;
		}

		i {
			font-size: 20px;
			margin-right: 4px;
		}
	}
}

.col {
	display: grid;
	grid-column-gap: 16px;
}

.col-1 {
	grid-template-columns: auto;
}

.col-2 {
	grid-template-columns: auto auto;
}
</style>
