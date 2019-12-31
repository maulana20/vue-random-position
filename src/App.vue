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
						<div class="card" v-if="!is_reset">
							<div class="card-header">
								<div class="card-header-title">Group</div>
							</div>
							<div class="card-content has-text-centered">
								<span class="is-size-4"><b>{{ group_member[group_index].group }}</b></span>
							</div>
						</div>
						<div v-else>
							<a id="export_excel"></a>
							<button class="button is-fullwidth is-danger" @click="reset()" style="margin-bottom: 5px;"><span class="is-size-8">RESET</span></button>
							<button class="button is-fullwidth is-info" @click="download()"><span class="is-size-8">EXPORT EXCEL</span></button>
						</div>
						<button class="button is-fullwidth is-info" v-if="!is_next && !is_reset" @click="random()"><span class="is-size-8">RANDOM</span></button>
						<button class="button is-fullwidth is-success" v-if="is_next && !is_reset" @click="next()"><span class="is-size-8">NEXT</span></button>
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
	import XLSX from 'xlsx';
	
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
			setRecord: function ()
			{
				var group_member = this.group_filter.group_member
				
				var record = [{}]
				
				group_member.map((data) => { return 'GROUP' + data.group }).forEach(function (group) { record[0][group] = group })
				for (var i = 0; i <= this.maximum; i++) { var obj = {}; group_member.forEach(function (data) { obj['GROUP'+data.group] = data.member[i] }); record.push(obj); }
				
				return record
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
			},
			download: function ()
			{
				var record = this.setRecord()
				var title = Object.keys(record[0]).map((data) => { return data })
				
				var cells = []
				record.forEach(function (data, index) {
					title.map((title, i) => { var obj = { position: String.fromCharCode('A'.charCodeAt(0) + i) + (index + 1), value: data[title] }; cells[obj.position] = { v: obj.value } })
				})
				
				// cells = [ A1: {v: "name"}, B1: {v: "size"}, C1: {v: "taste"}, D1: {v: "price"}, E1: {v: "remain"}, A2: {v: "rahmat karmawan"}, B2: {v: "12"}, C2: {v: "delicious"}, D2: {v: "40"}, E2: {v: "100"} ]
			
				let outputPos = Object.keys(cells)
				let tmpWB = { SheetNames: ['report'], Sheets: { 'report': Object.assign({}, cells, { '!ref': outputPos[0] + ':' + outputPos[outputPos.length - 1] }) } }
				let tmpDown = new Blob([this.s2ab(XLSX.write(tmpWB, { bookType: 'xlsx', bookSST: false, type: 'binary' } ))], { type: '' })
				
				
				// DONWLOAD
				document.getElementById('export_excel').download = 'vue_random_position.xlsx'
				document.getElementById('export_excel').href = URL.createObjectURL(tmpDown)
				document.getElementById('export_excel').click()
				
				setTimeout(() => URL.revokeObjectURL(tmpDown), 100)
			},
			s2ab: function (s)
			{
				var buf = new ArrayBuffer(s.length)
				
				var view = new Uint8Array(buf)
				for (var i = 0; i !== s.length; ++i) { view[i] = s.charCodeAt(i) & 0xFF }
				
				return buf
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
