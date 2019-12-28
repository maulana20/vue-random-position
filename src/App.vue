<template>
	<div class="container" id="wrapper">
		<div class="columns">
			<div class="column is-9">
				<div class="columns">
					<div class="column is-8">
						<div class="card">
							<div class="card-header">
								<div class="card-header-title">Country</div>
							</div>
							<div class="card-content">
								<h1 class="is-size-1">{{ country }}</h1>
							</div>
						</div>
					</div>
					<div class="column is-4">
						<div class="card">
							<div class="card-header">
								<div class="card-header-title">Group</div>
							</div>
							<div class="card-content has-text-centered">
								<span class="is-size-4"><b>{{ group_member[group_index].group }}</b></span>
							</div>
						</div>
						<button class="button is-fullwidth is-info" v-if="!is_next && !is_reset" @click="random()"><span class="is-size-8">RANDOM</span></button>
						<button class="button is-fullwidth is-success" v-if="is_next && !is_reset" @click="next()"><span class="is-size-8">NEXT</span></button>
						<button class="button is-fullwidth is-danger" v-if="is_reset" @click="reset()"><span class="is-size-8">RESET</span></button>
					</div>
				</div>
				<div class="columns" id="result-wrapper">
					<div class="column is-4" v-for="data in group_member">
						<div class="card">
							<div class="card-header">
								<h1 class="card-header-title">Group {{ data.group }}</h1>
							</div>
							<div class="card-content">
								<ol style="padding-left:10px;">
									<li style="margin-bottom:5px;" v-for="country in data.member">
										{{ country }}
									</li>
								</ol>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="column is-3">
				<div class="columns">
					<div class="column is-12">
						<div class="card">
							<div class="card-header">
								<h1 class="card-header-title">Participant</h1>
							</div>
							<div class="card-content">
								<ol style="padding-left:10px;">
									<li style="margin-bottom:5px;" v-for="country in country_list">
										{{ country }}
									</li>
								</ol>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data: function()
		{
			return {
				country_list: [
					'INDONESIA', 'MALAYSIA', 'SINGAPORE','THAILAND',
					'VIETNAM', 'THAILAND', 'PHILIPPINES', 'MYANMAR',
					'CAMBODIA', 'LAOS', 'TIMOR-LESTE', 'BRUNEI DARUSSALAM'
				],
				group_member: [
					{ group: 'A', member: [] },
					{ group: 'B', member: [] },
					{ group: 'C', member: [] }
				],
				group_index: 0,
				
				// FILTERING
				group_filter: { group_member: [], country_list: [], group_index: 0 },
				
				// SHOW IN DASHBOARD
				country: '-',
				maximum: 3,
				country_bin: [],
				
				disabled: false,
				is_next: false,
				is_reset: false,
			}
		},
		created: function()
		{
			if (this.group_filter.group_member.length <= 0) return
			
			this.group_member = this.group_filter.group_member
			this.country_list = this.group_filter.country_list
			this.group_index = this.group_filter.group_index
			
			if (this.country_list.length <= 0) this.is_reset = true
		},
		mounted: function ()
		{
			this.initGroup()
			this.initButton()
		},
		methods: {
			initGroup: function ()
			{
				this.group_member = this.group_member.map((data) => ({ group: data.group, member: [] }))
				
				this.group_index = 0
				
				this.group_filter.group_member = []
				this.group_filter.country_list = []
				this.group_filter.group_index = this.group_index
				
			},
			initButton: function ()
			{
				this.disabled = this.is_next = this.is_reset = false;
			},
			setGroup: function (group_member, country_list, group_index)
			{
				this.group_filter.group_member = group_member
				this.group_filter.country_list = country_list
				this.group_filter.group_index = group_index
			},
			getIndex: function ()
			{
				return Math.floor(Math.random() * this.country_list.length) + 0;
			},
			random: function ()
			{
				var index = this.getIndex()
				this.country = this.country_list[index]
				
				this.group_member[this.group_index].member.push(this.country)
				this.country_list = this.country_list.filter((country, i) => i !== index)
				
				this.country_bin.push(this.country)
				
				if (this.group_member[this.group_index].member.length > this.maximum) this.is_next = true
				if ((this.group_index + 1) === this.group_member.length && this.group_member[this.group_index].member.length > this.maximum) this.is_reset = true
				
				this.setGroup(this.group_member, this.country_list, this.group_index)
			},
			next: function ()
			{
				if (this.group_index < this.group_member.length) {
					this.group_index += 1
					this.country = '-'
					this.is_next = false
				}
			},
			reset: function ()
			{
				this.country = '-'
				
				this.country_list = this.country_bin
				this.country_bin = []
				
				this.initGroup()
				this.initButton()
			}
		}
	}
</script>
<style>
html { height: 100%; }
body { height: 100%; background-color: #ccd; }

#wrapper { padding-top: 20vh; }
#result-wrapper { padding-top: 10vh; }
</style>
