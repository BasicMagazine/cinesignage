<template>
  <div id="app">
  <div id="loading" v-show="loading">
	 <div class="loading">
		 <p>{{loading_txt}}</p>
	 </div>
  </div>
  <div id="overlay" v-show="overlay"></div>
  <header class="sticky-top" role="banner">
	  <Header :Data_calendar="Data_calendar" :area="area" @callParent_rearea="get_data" @callParent_reload="get_coords" @callParent_refresh="refresh_data" :data_exist="data_exist" :obj_count="results.length"></Header>
  </header>
  <main id="main_container" class="container-fluid" role="main">
	  <div id="changing" v-show="changing">
		  <div class="changing">
			  <p>読み込み中...</p>
		  </div>
	  </div>
	  <div id="result_nodata" v-if="results.length == 0">
		  <div class="result_nodata">
			  <p v-if="category == 'titles'">条件に合った作品はありません</p>
			  <p v-else-if="response_data == null">データを受信できませんでした</p>
			  <p v-else>条件に合った劇場はありません</p>
		  </div>
	  </div>
	  <Main :result_data="results" :gps_parent="gps"></Main>
	  <Modal @callParent_rearea="get_data" :modalView="modalShow"></Modal>
  </main>
  <footer class="fixed-bottom d-flex align-items-center table-responsive">
	  <Footer :Footer_list="Footer_data" :filter_num_parent="filter_num" @callParent_refresh="refresh_data"></Footer>
  </footer>
  </div>
 
</template>

<script>
import Header from './components/Header'
import Main from './components/Main'
import Modal from './components/Modal'
import Footer from './components/Footer'
import axios from 'axios'
import moment from 'moment'
moment.locale('ja')
import localarea from './assets/local_area.json'
import salesfilter from './assets/sales_filter.json'

export default {
  name: 'App',
  components: {
    Header,
	Main,
	Modal,
	Footer
  },
  data () {
    return {
		theater_data:null,
		area:{
			"pref":null,
			"current_location":null,
			"select_area":null,
			"location_txt":null
		},
		pref_code:null,
		geo:{
			"lat":null,
			"lng":null
		},
		time:{
			"day":moment().date(),
			"week":moment().weekday(),
			"day_select":moment().format('MMDD'),
			"unixtime":moment().startOf('day').unix()
		},
		category:"theater",
		response_data:null,
		data_exist:true,
		results:[],
		Data_calendar:[],
		Data_calendar_parent:[],
		Footer_data:[],
		filter_num:null,
		sales_filter:[],
		loading:true,
		loading_txt:null,
		overlay:false,
		changing:false,
		gps:false,
		modalShow:false
    }
  },
  created() {
	  if (!navigator.onLine) {
		  alert('オフラインになっています。電波状況が良い場所で再度お試しください。');
		  this.loading = false;
	  } else {
		  this.get_coords();
	  }  
  },
  methods: {
 	  get_coords() {
		  var options = {
			enableHighAccuracy: true,
			timeout: 8000,
			maximumAge: 0
          };
		  if (navigator.geolocation) {
			  this.loading_txt = "現在地を取得しています...";
			  navigator.geolocation.getCurrentPosition(
				  function(position){
					  let coords = position.coords;
					  // 緯度経度だけ取得
					  this.geo.lat = coords.latitude;
					  this.geo.lng = coords.longitude;
					  this.gps = true;
					  this.get_data();
				  }.bind(this),
				  function(error) {
					  console.log(error);
					  this.gps = false;
					  this.loading = false;
					  this.modalShow = true;
					  // エラー処理を書く
				  }.bind(this)
		  ,options);
		  } else {
			  // エラー処理を書く
			  console.log("err");
			  this.modalShow = true;
		  }
	  }, 
	  get_data() {
		  axios.get('https://36g9smua36.execute-api.ap-northeast-1.amazonaws.com/dev', {
			  params: {
				  lat: this.geo["lat"],
				  lng: this.geo["lng"],
				  category:this.category,
				  day:this.time["day"],
				  week:this.time["week"]
			  }
		  }).then(response => {
			  if(this.gps) {
				  this.area.pref = response.data.pref;
				  this.area.current_location = response.data.area;
				  this.area.location_txt = response.data.area;
			  }
			  this.loading_txt = "データを受信しています...";
			  this.pref_code = response.data.pref_code;
			  this.response_data = response.data.body;
			  console.log(response.data.body);
			  this.Data_calendar_parent = [];
			  this.refresh_data();
		  })
		   .catch(error => {
			   alert('データの取得に失敗しました。しばらくしてから再度お試しください');
			   this.data_exist = false;
			   this.area.current_location = "No_Data";
			   this.loading = false;
		   });
	  },
	  refresh_data() {
		if(!this.loading) {
			this.changing = true;
		}
		scrollTo(0, 0);
		
		var results = this.response_data;
		//データのフィルタリング
		var filter_num = this.filter_num;
		if(filter_num !== null) {
			var sort_filter = results.filter(function(item, index){
				if (item.address.area_code == filter_num) {
					return item;
				}
			});
			results = sort_filter;
		}
			
		//ヘッダーのカレンダーデータ作成
		if(this.Data_calendar.length == 0) {
			this.Data_calendar = make_data_calendar(results,this.category,this.sales_filter,this.Data_calendar_parent);
			if(this.Data_calendar_parent.length == 0) {
				this.Data_calendar_parent = this.Data_calendar;
			}
		}
		
		//カテゴリごとにソート
		switch (this.category){
			case "theater":
				results.sort(function(a, b){
					return (a.distance < b.distance) ? -1 : 1;
				});
				break;
			case "titles":
				results = sort_title(results);
				results.sort( compare );
				results = sort_titles( results );
				break;
			case "sales":
				var day = this.time["day"];
				var week = this.time["week"];
				var filter_arr = this.sales_filter;
				results = sort_sales(filter_arr,day,week,results );
				break;
		}
				
		//フッターのデータ作成		
		this.Footer_data = make_footer_data(localarea,this.category,this.pref_code);
		
		this.results = results;
		this.loading = false;
		if(this.changing) {
			setTimeout(() => {
				this.changing = false;
			},1500);
		}
			
		//↓処理用関数//

		//タイトル基準に並び替え
		function sort_title(movie_item) {
			
			var result = [];
			var titles = [];
			for (let i = 0; i < movie_item.length; i++) {
				var theater = movie_item[i]["theater"];
				for (let j = 0; j < movie_item[i]["schedule"].length; j++) {
					let title = movie_item[i]["schedule"][j]["title"];
					let release = movie_item[i]["schedule"][j]["release"];
					if ( titles.indexOf(title) == -1) {
						titles.push(title);
						let json = {
							"title":title,
							"release":release,
							"schedule":[]
						};
						result.push(json);
					} 
				}
			}
			
			for (let i = 0; i < result.length; i++) {
				var title = result[i]["title"];
				for (let j = 0; j < movie_item.length; j++) {
					let theater =  movie_item[j]["theater"];
					let schedules =  movie_item[j]["schedule"];
					let distance =  movie_item[j]["distance"];
					for (let k = 0; k < schedules.length; k++) {
						if(title == schedules[k]["title"]) {
							let json = {
								"theater":theater,
								"address":movie_item[j]["address"],
								"distance":distance,
								"time":schedules[k]["time"]
							};
							json.address["distance"] = distance;
							result[i]["schedule"].push(json);
						}
					}
				}
				result[i]["schedule"].sort(function(a, b){
					return (a.distance < b.distance) ? -1 : 1;
				});	    	
			}
			return result;
		}
		
		//割引劇場のソート
		function sort_sales(filter_arr,day,week,movie_item) {
			var sort_sales = movie_item.filter(function(item, index){
				var check;
				var sales_txt;
				if (item.sales.day[day] || item.sales.week[week]) {
					if(item.sales.day[day] && !item.sales.week[week]) {
						sales_txt = item.sales.day[day]["detail"];
						check = check_sales(filter_arr,sales_txt);
						if(check) {
							item["sales_txt"] = sales_txt;
						}
					} else if (!item.sales.day[day] && item.sales.week[week]) {
						sales_txt = item.sales.week[week]["detail"];
						check = check_sales(filter_arr,sales_txt);
						if(check) {
							item["sales_txt"] = sales_txt;
						}
					} else if(item.sales.day[day] && item.sales.week[week]) {
						sales_txt = item.sales.day[day]["detail"] + "," + item.sales.week[week]["detail"];
						check = check_sales(filter_arr,sales_txt);
						if(check) {
							item["sales_txt"] = sales_txt;
						}
					}
					if(check) {
						return item;
					}			   
				}
			});	
			
			sort_sales.sort(function(a, b){
				return (a.distance < b.distance) ? -1 : 1;
			});
		
			return sort_sales;
		}
	
		function check_sales(filter_arr,sales) {
			var filter_sales_list = [];
			for(var i = 0; i < filter_arr.length; i++){
				let num = filter_arr[i];
				let filter_txt = salesfilter[num].filter;
				filter_sales_list.push(filter_txt);
			}
			var result = true;
			for(var i = 0; i < filter_sales_list.length; i++){
				let filter = filter_sales_list[i];
				if ( sales.indexOf(filter) != -1) {
					result = false;
					break;
				} 
			}
			return result;
		}
	
		//公開日順ソート
		function compare( a, b ){
			var r = 0;
			if( a.release > b.release ){ r = -1; }
			else if( a.release < b.release ){ r = 1; }
			return r;
		}
		
		//同タイトルソート
		function sort_titles( body ){
			var return_arr = [];
			var release_arr = [];
			body.forEach((Obj) => {
				release_arr.push(Obj.release);
			});
			let set = new Set(release_arr);
			release_arr = Array.from(set);
			release_arr.forEach((release) => {
				var same_release = body.filter(function(item, index){
					if (item.release == release) return true;
				});
				var title_arr = []
				same_release.forEach((item) => {
					let title = item.title;
					if(!title) {
						return;
					}
						
					if ( title.indexOf(' [data]') != -1) {
						let searchIndex = title.lastIndexOf(" [data]");
						title = title.slice(0,searchIndex);
						title_arr.push(title);
					} else {
						title_arr.push(title);
					}
				});
				let set = new Set(title_arr);
				title_arr = Array.from(set);
		
				title_arr.forEach((title) => {
					let tmp_arr = same_release.filter(function(item, index){
						if ((item.title).indexOf(title) >= 0) return true;
					});	
					return_arr = return_arr.concat(tmp_arr);
				});
			});
			return return_arr;
		}
		
		//Header用カレンダーデータ作成
		function make_data_calendar(response,category,sales_filter,Data_calendar_parent) {
			var data_calendar = [];
			var sales_txt;
			if(category === "sales") {
				for (var i = 0; i < 5; i++) {
					var today = moment().add(i,'days').format('MMDD');
					if (Data_calendar_parent.indexOf(today) == -1) {
						continue;
					}
					var day_num = moment().add(i,'days').format('D');
					var week_num = moment().add(i,'days').format('d');
					response.forEach((data) => {
						if (data.sales.day[day_num] || data.sales.week[week_num]) {
							if (data_calendar.indexOf(today) == -1){
								var result = true;
								if( sales_filter.length > 0 ) {
									var sales_txt;
									if(data.sales.day[day_num] && !data.sales.week[week_num]) {
										sales_txt = data.sales.day[day_num]["detail"];
									} else if (!data.sales.day[day_num] && data.sales.week[week_num]) {
										sales_txt = data.sales.week[week_num]["detail"];
									} else if(data.sales.day[day_num] && data.sales.week[week_num]) {
										sales_txt = data.sales.day[day_num]["detail"] + "," + data.sales.week[week_num]["detail"];
									}
									console.log(sales_filter.length);
									for (var j = 0; j < sales_filter.length; j++) {
										let num = sales_filter[j];
										var filter_txt = salesfilter[num].filter;
										console.log(filter_txt);
										console.log(sales_txt);
										if (sales_txt.indexOf(filter_txt) != -1){
											result = false;
											break;
										}										
									}										
								}
								if(result) {
									data_calendar.push(today);
								}
							}				
						}
					});	
				}
			} else {
				var count = 0;
				var data_length = response.length;
				var day_arr = [];
				for (var i = 0; i < 5; i++) {
					day_arr.push(moment().add(i,'days').format('MMDD'));
				}
				
				response.forEach((data) => {
					var day_arr_tmp = [];
					data.schedule.forEach((schedule) => {
						schedule.time.forEach((time) => {
							let keys = Object.keys(time);
							if (time[keys[0]][0] != "no_play" && day_arr_tmp.indexOf(keys[0]) == -1){
								day_arr_tmp.push(keys[0]);
							}
						});	
					});
					
					var all_arr = [...day_arr, ...day_arr_tmp];
					
					var sortArr = all_arr.filter(
					  item => day_arr.includes(item) && day_arr_tmp.includes(item)
					);
					
					day_arr = [...new Set(sortArr)];
									
				});
				data_calendar = day_arr;
			}
			
			return data_calendar;
		}

		//Footer用データ作成
		function make_footer_data(footer_source,category,pref_code) {
			var arr = [];
			let local_area = footer_source[pref_code];
			let count = Object.keys(local_area).length;
			arr = ["すべて"];
			for (var i = 0; i < count; i++){
				arr.push(local_area[i]);
			 }
			 return arr;
		 }
		 		 
	//↑処理用関数//
	} 
  }
}

</script>


