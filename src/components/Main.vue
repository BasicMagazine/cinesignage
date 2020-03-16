<template>
<div>	

	<div class="lists_cinema" v-for="(result,index) in display_data" v-bind:key="result.id">
    	<div class="theaterName d-flex align-items-center" v-bind:class="{ disable: result.disabled === true }">
		  <div class="row">
			  <div v-if="result.category == 'titles'">
				  <h2 class="col-12"> {{ result.heading_parent }}
				  <span class="screen_data" v-bind:class="{ disable: result.disabled === true }" v-for="screen_data in result.parent_data.screen_data" v-bind:key="result.parent_data.screen_data.id">{{ screen_data }}</span>
				  </h2>
				  <p class="col-12">{{ result.parent_data.release }}</p>
				  <p class="col-12">{{ result.schedules.length }}館で上映中</p>
				  <div class="info col-12">
					  <b-button class="schedules"
					  	:class="[visible[index] ? null : 'collapsed',{open: visible[index] === true}]"
      					:aria-expanded="visible[index] ? 'true' : 'false'"
      					:aria-controls="'collapse-' + index"
      					v-on:click="collapseClick(index,result.disabled)"	  
					  >上映スケジュール</b-button>
				  </div>
			  </div>	  
			  <div v-else>
				  <h2 class="col-12">
				  <span class="caution" v-if="result.status"> {{ result.status }}</span>
				  {{ result.heading_parent }}&nbsp;
				  <span v-if="gps">&nbsp;現在地より約{{ Math.floor( result.parent_data.address.distance / 1000 * Math.pow( 10, 1 ) ) / Math.pow( 10, 1 ) }}km</span>
				  </h2>
				  <div v-if="result.category == 'sales'">
					  <p class="col-12">{{ result.parent_data.sales_txt }}</p>
				  </div>
				  <p class="col-12">{{ result.parent_data.address.map }}</p>
			      <div class="info col-12">
					  <b-button class="schedules"
					  	:class="[visible[index] ? null : 'collapsed',{open: visible[index] === true}]"
      					:aria-expanded="visible[index] ? 'true' : 'false'"
      					:aria-controls="'collapse-' + index"
      					v-on:click="collapseClick(index,result.disabled)"	  
					  >上映スケジュール</b-button>
					  <b-button v-bind:href="'https://maps.google.co.jp/maps/search/' + result.parent_data.address.map" target="_blank">MAP</b-button>
					  <b-button v-bind:href="result.parent_data.address.url" target="_blank">URL</b-button>
					  <b-button v-bind:href="'tel:' + result.parent_data.address.tel">TEL</b-button>
			      </div>
			  </div>
		  </div>
    	</div>
      <b-collapse v-bind:id="'collapse-' + index" v-model="visible[index]">
        <div class="item_cinema">
			<div v-if="result.status == '上映データなし'" class="no_result d-flex align-items-center">
				<p>上映スケジュールが存在しません。直接劇場にお尋ねください</p>
			</div>
			<div v-else class="row" v-for="cinema in result.schedules" v-bind:key="cinema.id">
				<div class="item_left col-md-5 col-xs-12 d-flex align-items-center">
					<div class="inner">
						<h3>{{ cinema.heading_child }}</h3>
						 <div v-if="result.category == 'titles'">
							 <ul class="list-inline inner_titles">
								<a v-bind:href="'https://maps.google.co.jp/maps/search/' + cinema.child_data.map" target="_blank"><li class="list-inline-item">MAP</li></a>
								<a v-bind:href="cinema.child_data.url" target="_blank"><li class="list-inline-item">URL</li></a>
								<a v-bind:href="'tel:' + cinema.child_data.tel"><li class="list-inline-item">TEL</li></a>
							</ul>
						 </div>
						 <div v-else>
							<ul class="list-inline">
								<li class="list-inline-item cinema_data" v-for="child_data in cinema.child_data" v-bind:key="child_data.id">{{ child_data }}</li>
							</ul>
						</div>
					</div>
				</div>
				<div v-if="cinema.time_exist" class="item_right col-md-7 col-xs-12">
					<ul class="list-inline">
						<li class="list-inline-item" v-for="times in cinema.time" v-bind:key="times.id"><p v-bind:class="{ 'show-active' : times.s }">{{ times.t }}</p></li>
					</ul>
				</div>
				<div v-else class="item_right col-md-7 col-xs-12 d-flex align-items-center caution">
					<p>{{cinema.time_caution}}</p>
				</div>
			</div>
        </div>
      </b-collapse>
	</div>
	
</div>
</template>

<script>
import moment from 'moment';
moment.locale('ja');

export default {
	data () {
		return {
			display_data:null,
			visible:[],
			gps:false,
			accordion_num:null
		}
	},
	props: {
		result_data: null,
		gps_parent: null
	},
	mounted () {
    	setInterval(()=>{
			if (this.display_data !== null) {
				var now_time = moment().format('YYYY-MM-DD HH:mm');
				var day = this.$parent.time["day_select"];
				this.display_data.forEach((data) => {
					data["schedules"].forEach((schedule) => {
						schedule["time"].forEach((time) => {
							if (time["t"].length > 1) {
								let time_data = moment().format(`YYYY-MM-DD ${time["t"]}`);
								if(day == moment().format('MMDD') && moment(time_data).isBefore(now_time)) {
									time['s'] = false;
								}
							}
						});
					});
					if(data["schedules"].length > 0 && !data.status) {
						var count = 0;
						data["schedules"].forEach((schedules) => {
							schedules["time"].forEach((time) => {
								if(time["s"]) {
									count ++
								}
							});
						});
						if(count == 0) {
							data.disabled = true;
						} else {
							data.disabled = false;
						}
					}
				});
			}
		},30000);
	},
	methods: {
		collapseClick(index) {
				
			let visible = this.visible[index];
			this.visible.splice(index, 1);
			
			if(!visible) {
				this.visible.splice(index, 0, true);
			} else {
				this.visible.splice(index, 0, false);
			}
		}
	},
    watch: {
      result_data: function (val, oldVal) {
	  	this.visible = [];
	  	
		for (var i = 0; i < val.length; i++) {
			this.visible.push(false);
		}
		
 		var result_arr = [];
		var now_time = moment().format('YYYY-MM-DD HH:mm');
		var day = this.$parent.time["day_select"];
		val.forEach((value) => {
			var result = {};
			result.disabled = true;
			result.category = this.$parent.category;
			result.status = false;
			let heading_parent;
			let parent_data = {};
			if(this.$parent.category == "titles") {
				heading_parent = value.title;
				if ( heading_parent.indexOf(' [data]') != -1) {
					let data = screen_data(heading_parent);
					heading_parent = data.title;
					parent_data.screen_data = data.screen_data;
				}
				
				let release = moment.unix(value.release).format('YYYY年MM月DD日全国公開');
				parent_data.release = release;
			} else {
				heading_parent = value.theater;
				parent_data.address = value.address;
				parent_data.address["distance"] = value.distance;
				if(this.$parent.category == "sales") {
					parent_data.sales_txt = value.sales_txt;
				}
			}
			result.heading_parent = heading_parent;
			result.parent_data = parent_data;
			var schedules_arr = [];
			value["schedule"].forEach((schedules,index) => {
				let schedule = {};
				let heading_child;
				let child_data;
				if(this.$parent.category == "titles") {
					heading_child = schedules.theater;
					child_data = schedules.address
				} else {
					heading_child = schedules.title;
					if ( heading_child && heading_child.indexOf(' [data]') != -1) {
						let data = screen_data(heading_child);
						heading_child = data.title;
						child_data = [];
						child_data = data.screen_data;
					} else {
						child_data = null;
					}
				}
				schedule.heading_child = heading_child;
				schedule.child_data = child_data;
				
				var time_arr = false;
				schedules["time"].forEach((time) => {
					let keys = Object.keys(time);
					if(keys == day && time[keys][0]) {
						var push_data = true;
						switch (time[keys][0]){
							case "no_play":
								push_data = false;
								break;
							case "no_schedule":
								schedule.time = [];
								schedule.time_exist = false;
								schedule.time_caution = "上映データなし";						
								break;
							case "no_time":
								schedule.time = [];
								schedule.time_exist = false;
								schedule.time_caution = "スケジュールなし";						
								break;
							case "close":
								schedule.time = [];
								schedule.time_exist = false;
								schedule.time_caution = "本日休館";
								console.log(schedule);
								break;
							default:
								time_arr = [];
								for (var i = 0; i < time[keys].length; i++) {
									let time_data = moment().format(`YYYY-MM-DD ${time[keys][i]}`);
									let data = {
										't':time[keys][i],
										's':true
									};
									if(day == moment().format('MMDD') && moment(time_data).isBefore(now_time)) {
										data['s'] = false;
									}
									time_arr.push(data);
								}
								let count = 10 - time_arr.length;
								for (var i = 0; i < count; i++) {
									let data = {
										't':"\u00A0",
										's':false
									};
									time_arr.push(data);
								}
								schedule.time = time_arr;
								schedule.time_exist = true;
								schedule.time_caution = false;
								break;
						}
						if(push_data) {
							schedules_arr.push(schedule);
						}
					}
				});
				
			});
			
			var has_schedule = false;
			
			var caution_data = { "no_play":0, "no_data":0, "no_schedule":0,"close":0 };
			
			schedules_arr.forEach((schedules) => {
				if(schedules["time_exist"]) {
					has_schedule = true;
				} else {
					switch (schedules["time_caution"]){
						case "上映なし":
							caution_data["no_play"]++;
							break;
						case "上映データなし":
							caution_data["no_data"]++;
							break;
						case "スケジュールなし":
							caution_data["no_schedule"]++;
							break;
						case "本日休館":
							caution_data["close"]++;
							break;
					}
				}
			});	
			
			if(!has_schedule) {
				let arr_count = schedules_arr.length;
				let text;
				if(caution_data["no_play"] == arr_count) {
					text = "上映作品なし";
				} else if(caution_data["no_data"] == arr_count) {
					text = "上映データなし";
				} else if(caution_data["no_schedule"] == arr_count) {
					text = "上映スケジュールなし";
				} else if(caution_data["close"] == arr_count) {
					text = "本日休館";
				}
				result.status = text;
				result.disabled = true;
			}
			
			var time_count = 0;
			if(schedules_arr.length > 0) {
				var time_switch = false;
				schedules_arr.forEach((schedules) => {
					if(!schedules.time_exist) {
						time_switch = true;
						return;
					} else {
						schedules["time"].forEach((time) => {
							if(time["s"]) {
								time_count++;
								time_switch = true;
								result.disabled = false;
								return;
							}
						});
					}
				});
				
				
				result.schedules = schedules_arr;
				if(!time_switch || time_count == 0) {
					if(!result.status) {
						result.status = "本日上映終了";
					}
				} 
				result_arr.push(result);
				
			} else {
				result.schedules = [];
				result.disabled = true;
				if(this.$parent.category !== "titles") {
					result_arr.push(result);
				}
			}
			
			});
			
			console.log(result_arr);
						
			this.display_data = result_arr;
			this.gps = this.gps_parent;
						
			function screen_data(titles) {
				var data = {
					title:String,
					screen_data:[]
				};
				let searchIndex = titles.lastIndexOf(" [data]");
				data.title = titles.slice(0,searchIndex);
				var screen_data_txt = titles.slice(searchIndex,-1);
				screen_data_txt = screen_data_txt.replace("[data] (","");
				data.screen_data = screen_data_txt.split(",");
				return data;
			}
      }
    }
}
</script>