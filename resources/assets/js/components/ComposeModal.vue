<template>
	<div>
		<input type="file" name="media" class="d-none file-input" multiple="" v-bind:accept="config.uploader.media_types">
		<div class="timeline">
			<div class="card status-card card-md-rounded-0">
				<div class="card-header d-inline-flex align-items-center bg-white">
					<img v-bind:src="profile.avatar" width="32px" height="32px" style="border-radius: 32px;" class="box-shadow">
					<a class="username font-weight-bold pl-2 text-dark" v-bind:href="profile.url">
						{{profile.username}}
					</a>
					<div class="text-right" style="flex-grow:1;">
						<div class="dropdown">
							<button class="btn btn-link text-dark no-caret dropdown-toggle" type="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" title="Post options">
								<span class="fas fa-ellipsis-v fa-lg text-muted"></span>
							</button>
							<div class="dropdown-menu dropdown-menu-right" aria-labelledby="dropdownMenuButton">
								<div v-show="media.length > 0" class="dropdown-item small font-weight-bold" v-on:click="mediaDrawer = !mediaDrawer">{{mediaDrawer ? 'Hide' : 'Show'}} Media Toolbar</div>
								<div class="dropdown-item small font-weight-bold" v-on:click="about">About</div>
								<a class="dropdown-item small font-weight-bold" href="/i/compose">Classic Compose UI</a>
								<div class="dropdown-divider"></div>
								<div class="dropdown-item small font-weight-bold" v-on:click="closeModal">Close</div>
							</div>
						</div>
					</div>
				</div>

				<div class="postPresenterContainer">
					<div v-if="ids.length == 0" class="w-100 h-100 bg-light py-5 cursor-pointer" style="border-bottom: 1px solid #f1f1f1" v-on:click="addMedia()">
						<p class="text-center mb-0 font-weight-bold p-5">Click here to add photos.</p>
					</div>
					<div v-if="ids.length > 0">
						
						<b-carousel id="p-carousel"
							style="text-shadow: 1px 1px 2px #333;"
							controls
							indicators
							background="#ffffff"
							:interval="0"
							v-model="carouselCursor"
						>
							<b-carousel-slide  v-if="ids.length > 0" v-for="(preview, index) in media" :key="'preview_media_'+index">
								<div slot="img" :class="[media[index].filter_class?media[index].filter_class + ' cursor-pointer':' cursor-pointer']" v-on:click="addMedia()">
									<img class="d-block img-fluid w-100" :src="preview.url" :alt="preview.description" :title="preview.description">
								</div>
							</b-carousel-slide>
						</b-carousel>
					</div>
					<div v-if="mediaDrawer" class="bg-dark align-items-center">
						<ul class="nav media-drawer-filters text-center">
							<li class="nav-item">
								<div class="p-1 pt-3">
									<img :src="media[carouselCursor].url" width="100px" height="60px" v-on:click.prevent="toggleFilter($event, null)" class="cursor-pointer">
								</div>
								<a :class="[media[carouselCursor].filter_class == null ? 'nav-link text-white active' : 'nav-link text-muted']" href="#" v-on:click.prevent="toggleFilter($event, null)">No Filter</a>
							</li>
							<li class="nav-item" v-for="(filter, index) in filters">
								<div class="p-1 pt-3">
									<div :class="filter[1]" v-on:click.prevent="toggleFilter($event, filter[1])">
										<img :src="media[carouselCursor].url" width="100px" height="60px" class="">
									</div>
								</div>
								<a :class="[media[carouselCursor].filter_class == filter[1] ? 'nav-link text-white active' : 'nav-link text-muted']" href="#" v-on:click.prevent="toggleFilter($event, filter[1])">{{filter[0]}}</a>
							</li>
						</ul>
					</div>
					<div v-if="mediaDrawer" class="bg-lighter p-2 row">
						<div class="col-12">
							<div class="form-group">
								<input type="text" class="form-control" v-model="media[carouselCursor].alt" placeholder="Optional image description">
							</div>

							<div class="form-group">
								<input type="text" class="form-control" v-model="media[carouselCursor].license" placeholder="Optional media license">
							</div>
						</div>
						<!-- <div class="col-6 pt-2">
							<button class="btn btn-outline-secondary btn-sm mr-1"><i class="fas fa-map-marker-alt"></i></button>
							<button class="btn btn-outline-secondary btn-sm"><i class="fas fa-tools"></i></button>
						</div> -->
						<div class="col-12 text-right pt-2">
							<button class="btn btn-outline-danger btn-sm font-weight-bold mr-1" v-on:click="deleteMedia()">Delete Photo</button>
							<button class="btn btn-outline-secondary btn-sm font-weight-bold" v-on:click="updateMedia()">Hide Media Toolbar</button>
						</div>
					</div>
				</div>

				<div :class="[mediaDrawer?'d-none':'card-body']">
					<div class="caption">
						<p class="mb-2">
							<textarea class="form-control d-inline-block" rows="3" placeholder="Add an optional caption" v-model="composeText"></textarea>
						</p>
					</div>
					<div class="comments">
					</div>
					<div class="timestamp pt-1">
						<p class="small text-uppercase mb-0">
							<span class="text-muted">
								Draft
							</span>
						</p>
					</div>
				</div>

				<div :class="[mediaDrawer?'d-none':'card-footer']">
					<div class="d-flex justify-content-between align-items-center">
						<div>
							<div class="custom-control custom-switch d-inline mr-3">
								<input type="checkbox" class="custom-control-input" id="nsfwToggle" v-model="nsfw">
								<label class="custom-control-label small font-weight-bold text-muted pt-1" for="nsfwToggle">NSFW</label>
							</div>
							<div class="dropdown d-inline">
								<button class="btn btn-outline-secondary btn-sm py-0 dropdown-toggle" type="button" id="visibility" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
									{{visibility[0].toUpperCase() + visibility.slice(1)}}
								</button>
								<div class="dropdown-menu" aria-labelledby="visibility" style="width: 200px;">
									<a :class="[visibility=='public'?'dropdown-item active':'dropdown-item']" href="#" data-id="public" data-title="Public" v-on:click.prevent="visibility = 'public'">
										<div class="row">
											<div class="d-none d-block-sm col-sm-2 px-0 text-center">
												<i class="fas fa-globe"></i>
											</div> 
											<div class="col-12 col-sm-10 pl-2">
												<p class="font-weight-bold mb-0">Public</p>
												<p class="small mb-0">Anyone can see</p>
											</div> 
										</div>
									</a>
									<a :class="[visibility=='private'?'dropdown-item active':'dropdown-item']" href="#" data-id="private" data-title="Followers Only" v-on:click.prevent="visibility = 'private'">
										<div class="row">
											<div class="d-none d-block-sm col-sm-2 px-0 text-center">
												<i class="fas fa-lock"></i>
											</div> 
											<div class="col-12 col-sm-10 pl-2">
												<p class="font-weight-bold mb-0">Followers Only</p>
												<p class="small mb-0">Only followers can see</p>
											</div> 
										</div>
									</a>
									<a :class="[visibility=='private'?'dropdown-item active':'dropdown-item']" href="#" data-id="private" data-title="Followers Only" v-on:click.prevent="visibility = 'unlisted'">
										<div class="row">
											<div class="d-none d-block-sm col-sm-2 px-0 text-center">
												<i class="fas fa-lock"></i>
											</div> 
											<div class="col-12 col-sm-10 pl-2">
												<p class="font-weight-bold mb-0">Unlisted</p>
												<p class="small mb-0">Not listed on public timelines</p>
											</div> 
										</div>
									</a>
									<!-- <a class="dropdown-item" href="#" data-id="circle" data-title="Circle">
										<div class="row">
											<div class="col-12 col-sm-2 px-0 text-center">
												<i class="far fa-circle"></i>
											</div> 
											<div class="col-12 col-sm-10 pl-2">
												<p class="font-weight-bold mb-0">Circle</p>
												<p class="small mb-0">Select a circle</p>
											</div> 
										</div>
									</a>
									<a class="dropdown-item" href="#" data-id="direct" data-title="Direct Message">
										<div class="row">
											<div class="col-12 col-sm-2 px-0 text-center">
												<i class="fas fa-envelope"></i>
											</div> 
											<div class="col-12 col-sm-10 pl-2">
												<p class="font-weight-bold mb-0">Direct Message</p>
												<p class="small mb-0">Recipients only</p>
											</div> 
										</div>
									</a> -->
								</div>
							</div>
						</div>
						<div class="small text-muted font-weight-bold">
							{{composeText.length}} / 500
						</div>
						<div class="pl-md-5">
							<div class="btn-group">
								<button type="button" class="btn btn-primary btn-sm font-weight-bold" v-on:click="compose()">{{composeState[0].toUpperCase() + composeState.slice(1)}}</button>
								<button type="button" class="btn btn-primary btn-sm dropdown-toggle dropdown-toggle-split" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
									<span class="sr-only">Toggle Dropdown</span>
								</button>
								<div class="dropdown-menu dropdown-menu-right">
									<a :class="[composeState == 'publish' ?'dropdown-item font-weight-bold active':'dropdown-item font-weight-bold ']" href="#" v-on:click.prevent="composeState = 'publish'">Publish now</a>
									<!-- <a :class="[composeState == 'draft' ?'dropdown-item font-weight-bold active':'dropdown-item font-weight-bold ']" href="#" v-on:click.prevent="composeState = 'draft'">Save as draft</a>
									<a :class="[composeState == 'schedule' ?'dropdown-item font-weight-bold active':'dropdown-item font-weight-bold ']" href="#" v-on:click.prevent="composeState = 'schedule'">Schedule for later</a>
									<div class="dropdown-divider"></div>
									<a :class="[composeState == 'delete' ?'dropdown-item font-weight-bold active':'dropdown-item font-weight-bold ']" href="#" v-on:click.prevent="composeState = 'delete'">Delete</a> -->
								</div>
							</div>
						</div>
					</div>
				</div>

			</div>
		</div>
	</div>
</template>

<style type="text/css" scoped>
	.media-drawer-filters {
		overflow-x: scroll;
		flex-wrap:unset;
	}
	.media-drawer-filters .nav-link {
		min-width:100px;
		padding-top: 1rem;
		padding-bottom: 1rem;
	}
	.media-drawer-filters .active {
		color: #fff;
		font-weight: bold;
	}
	.media-drawer-filters::-webkit-scrollbar {
	    display: none;
	}
</style>
<script type="text/javascript">
export default {
	data() {
		return {
			config: {
				uploader: {
					media_types: '',
				}
			},
			profile: {},
			composeText: '',
			composeTextLength: 0,
			nsfw: false,
			filters: [],
			ids: [],
			media: [],
			carouselCursor: 0,
			visibility: 'public',
			mediaDrawer: false,
			composeState: 'publish'
		}
	},

	beforeMount() {
		this.fetchConfig();
		this.fetchProfile();
	},

	mounted() {
		this.mediaWatcher();
		this.filters = [
			['1977','filter-1977'], 
			['Aden','filter-aden'], 
			['Amaro','filter-amaro'], 
			['Ashby','filter-ashby'], 
			['Brannan','filter-brannan'], 
			['Brooklyn','filter-brooklyn'], 
			['Charmes','filter-charmes'], 
			['Clarendon','filter-clarendon'], 
			['Crema','filter-crema'], 
			['Dogpatch','filter-dogpatch'], 
			['Earlybird','filter-earlybird'], 
			['Gingham','filter-gingham'], 
			['Ginza','filter-ginza'], 
			['Hefe','filter-hefe'], 
			['Helena','filter-helena'], 
			['Hudson','filter-hudson'], 
			['Inkwell','filter-inkwell'], 
			['Kelvin','filter-kelvin'], 
			['Kuno','filter-juno'], 
			['Lark','filter-lark'], 
			['Lo-Fi','filter-lofi'], 
			['Ludwig','filter-ludwig'], 
			['Maven','filter-maven'], 
			['Mayfair','filter-mayfair'], 
			['Moon','filter-moon'], 
			['Nashville','filter-nashville'], 
			['Perpetua','filter-perpetua'], 
			['Poprocket','filter-poprocket'], 
			['Reyes','filter-reyes'], 
			['Rise','filter-rise'], 
			['Sierra','filter-sierra'], 
			['Skyline','filter-skyline'], 
			['Slumber','filter-slumber'], 
			['Stinson','filter-stinson'], 
			['Sutro','filter-sutro'], 
			['Toaster','filter-toaster'], 
			['Valencia','filter-valencia'], 
			['Vesper','filter-vesper'], 
			['Walden','filter-walden'], 
			['Willow','filter-willow'], 
			['X-Pro II','filter-xpro-ii']
		];
	},

	methods: {

		fetchConfig() {
			axios.get('/api/v2/config').then(res => {
				this.config = res.data;
			});
		},

		fetchProfile() {
			axios.get('/api/v1/accounts/verify_credentials').then(res => {
				this.profile = res.data;
			}).catch(err => {
				console.log(err)
			});
		},

		addMedia() {
			let el = $(event.target);
			el.attr('disabled', '');
			let fi = $('.file-input[name="media"]');
			fi.trigger('click');
			el.blur();
			el.removeAttr('disabled');
		},

		mediaWatcher() {
			let self = this;
			$(document).on('change', '.file-input', function(e) {
				let io = document.querySelector('.file-input');
				Array.prototype.forEach.call(io.files, function(io, i) {
					if(self.media && self.media.length + i >= self.config.uploader.album_limit) {
						swal('Error', 'You can only upload ' + self.config.uploader.album_limit + ' photos per album', 'error');
						return;
					}
					let type = io.type;
					let acceptedMimes = self.config.uploader.media_types.split(',');
					let validated = $.inArray(type, acceptedMimes);
					if(validated == -1) {
						swal('Invalid File Type', 'The file you are trying to add is not a valid mime type. Please upload a '+self.config.uploader.media_types+' only.', 'error');
						return;
					}

					let form = new FormData();
					form.append('file', io);

					let xhrConfig = {
						onUploadProgress: function(e) {
							let progress = Math.round( (e.loaded * 100) / e.total );
						}
					};

					axios.post('/api/v1/media', form, xhrConfig)
					.then(function(e) {
						self.ids.push(e.data.id);
						self.media.push(e.data);
						setTimeout(function() {
							self.mediaDrawer = true;
						}, 1000);
					}).catch(function(e) {
						swal('Oops, something went wrong!', 'An unexpected error occurred.', 'error');
					});
					io.value = null;
				});
			});
		},

		toggleFilter(e, filter) {
			this.media[this.carouselCursor].filter_class = filter;
		},

		updateMedia() {
			this.mediaDrawer = false;
		},

		deleteMedia() {
			if(window.confirm('Are you sure you want to delete this photo?') == false) {
				return;
			}
			let id = this.media[this.carouselCursor].id;
			axios.delete('/api/v1/media', {
				params: {
					id: id
				}
			}).then(res => {
				if(this.media.length == 1) {
					this.mediaDrawer = false;
					this.ids = [];
					this.media = [];
					this.carouselCursor = 0;
				}
				this.ids.splice(this.carouselCursor, 1);
				this.media.splice(this.carouselCursor, 1);
			}).catch(err => {
				swal('Whoops!', 'An error occured when attempting to delete this, please try again', 'error');
			});
		},

		mediaAltText() {
			return;
			// deprecate 
			swal({
				text: 'Add a media description',
				content: "input"
			}).then(val => {
				let media = this.media[this.carouselCursor];
				media.alt = val;
			});

		},

		mediaLicense() {
			return;
			// deprecate
			swal({
				text: 'Add a media license',
				content: "input",
				button: {
					text: "Update",
					closeModal: true,
				},
			}).then(val => {
				let media = this.media[this.carouselCursor];
				media.license = val;
			});

		},

		compose() {
			let state = this.composeState;

			switch(state) {
				case 'publish' :
					if(this.media.length == 0) {
						swal('Whoops!', 'You need to add media before you can save this!', 'warning');
						return;
					}
					if(this.composeText == 'Add optional caption...') {
						this.composeText = '';
					}
					let data = {
						media: this.media,
						caption: this.composeText,
						visibility: this.visibility,
						cw: this.nsfw
					};
					axios.post('/api/v2/status/compose', data)
					.then(res => {
						let data = res.data;
						window.location.href = data;
					}).catch(err => {
						swal('Oops, something went wrong!', 'An unexpected error occurred.', 'error');
					});
					return;
				break;

				case 'delete' :
					this.mediaDrawer = false;
					this.ids = [];
					this.media = [];
					this.carouselCursor = 0;
					this.composeText = '';
					this.composeTextLength = 0;
					$('#composeModal').modal('hide');
					return;
				break;
			}
		},

		about() {
			let text = document.createElement('div');
			text.innerHTML = `
				<p class="small font-weight-bold">Please visit the <a href="/site/kb/sharing-media">Sharing Media</a> page for more info.</p>
			`;
			swal({
				title: 'Compose UI v3', 
				content: text, 
				icon: 'info'
			});
		},

		closeModal() {
			$('#composeModal').modal('hide');
		}
	}
}
</script>