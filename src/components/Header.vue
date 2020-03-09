<template>
<div>
<div id="overlay" v-show="overlay"></div>
<div class="display-info">
	<div class="row">
		<div class="info-item order-1 col-4">
			    <button type="button" class="drawer-toggle drawer-hamburger" v-on:click="setting()">
      <span class="sr-only">toggle navigation</span>
      <span class="drawer-hamburger-icon"></span>
    </button>
    <nav class="drawer-nav" role="navigation">
		<div class="container">
			<h1>劇場情報スケジューラー</h1>
			<div class="nav_area">
				<h2 class="area">表示エリア</h2>
				<b-button v-on:click="reloadClick(0)">現在地を再取得する</b-button>
				<b-form-select v-model="selected_area" :options="geo_list"></b-form-select>
			</div>
			<div class="nav_area">
				<h2 class="sales">表示しない割引</h2>
				<b-form-checkbox-group
					v-model="selected_sales"
					:options="sales_filter"
					stacked
					class="sales_form"
				></b-form-checkbox-group>
			</div>
		</div>
    </nav>	
		</div>
		<div class="info-item order-2 col-4">
			<div class="inner d-flex align-items-center">
				<p class="place mx-auto">{{ area.pref }}{{ area.city }}</p>
			</div>
		</div>
  		<div class="info-item order-3 col-4">
			<div class="inner d-flex align-items-center">
				<p class="time mx-auto">{{ date }}</p>
			</div>
		</div>
	</div>
</div>
<div class="display-menu">
	<div class="row">
		<div class="menu-item theater col-md-4 col-sm-4 col-4" v-bind:class="{ active: menu_button_index === 0 }" v-on:click="menubtnClick(0)">
			<div class="inner">
				<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><path d="M40,9H10a1,1,0,0,0-1,1V28a1,1,0,0,0,1,1h2a1,1,0,0,0,0-2H11V11H39V27H38a1,1,0,0,0,0,2h2a1,1,0,0,0,1-1V10A1,1,0,0,0,40,9Z" transform="translate(-9 -9)"/><path d="M22.409,25a1.416,1.416,0,0,0,.767-.225l7.179-4.6a1.384,1.384,0,0,0,0-2.339l-7.182-4.606a1.421,1.421,0,0,0-1.449-.05A1.392,1.392,0,0,0,21,14.4v9.21a1.392,1.392,0,0,0,.726,1.22A1.434,1.434,0,0,0,22.409,25ZM23,15.488,28.475,19,23,22.512Z" transform="translate(-9 -9)"/><path d="M17,26a3,3,0,0,0-3,3v2H13a1,1,0,0,0,0,2H37a1,1,0,0,0,0-2H36V29a3,3,0,0,0-6,0v2H28V29a3,3,0,0,0-6,0v2H20V29A3,3,0,0,0,17,26Zm-1,5V29a1,1,0,0,1,2,0v2Zm16-2a1,1,0,0,1,2,0v2H32Zm-8,0a1,1,0,0,1,2,0v2H24Z" transform="translate(-9 -9)"/><path d="M40,39V37a3,3,0,0,0-6,0v2H32V37a3,3,0,0,0-6,0v2H24V37a3,3,0,0,0-6,0v2H16V37a3,3,0,0,0-6,0v2a1,1,0,0,0,0,2H40a1,1,0,0,0,0-2ZM12,39V37a1,1,0,0,1,2,0v2Zm8,0V37a1,1,0,0,1,2,0v2Zm8,0V37a1,1,0,0,1,2,0v2Zm8,0V37a1,1,0,0,1,2,0v2Z" transform="translate(-9 -9)"/></svg>
				<p>劇場</p>
			</div>
		</div>
  		<div class="menu-item title col-md-4 col-sm-4 col-4" v-bind:class="{ active: menu_button_index === 1 }" v-on:click="menubtnClick(1)">
			<div class="inner">
				<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><path d="M34,9a1,1,0,0,0-1,1v1H17V10a1,1,0,0,0-2,0V23a1,1,0,0,0,1,1H34a1,1,0,0,0,1-1V10A1,1,0,0,0,34,9ZM17,13H33v9H17Z" transform="translate(-9 -9)"/><path d="M34,26H16a1,1,0,0,0-1,1V40a1,1,0,0,0,2,0V39H33v1a1,1,0,0,0,2,0V27A1,1,0,0,0,34,26ZM17,37V28H33v9Z" transform="translate(-9 -9)"/><path d="M40,9a1,1,0,0,0-1,1V40a1,1,0,0,0,2,0V10A1,1,0,0,0,40,9Z" transform="translate(-9 -9)"/><rect x="27" y="5" width="2" height="2"/><rect x="27" y="1" width="2" height="2"/><rect x="27" y="9" width="2" height="2"/><rect x="27" y="13" width="2" height="2"/><rect x="27" y="17" width="2" height="2"/><rect x="27" y="21" width="2" height="2"/><rect x="27" y="25" width="2" height="2"/><rect x="27" y="29" width="2" height="2"/><path d="M10,9a1,1,0,0,0-1,1V40a1,1,0,0,0,2,0V10A1,1,0,0,0,10,9Z" transform="translate(-9 -9)"/><rect x="3" y="5" width="2" height="2"/><rect x="3" y="1" width="2" height="2"/><rect x="3" y="9" width="2" height="2"/><rect x="3" y="13" width="2" height="2"/><rect x="3" y="17" width="2" height="2"/><rect x="3" y="21" width="2" height="2"/><rect x="3" y="25" width="2" height="2"/><rect x="3" y="29" width="2" height="2"/></svg>
				<p>作品</p>
			</div>
		</div>
  		<div class="menu-item sales col-md-4 col-sm-4 col-4" v-bind:class="{ active: menu_button_index === 2 }" v-on:click="menubtnClick(2)">
			<div class="inner">
				<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><rect x="7" y="12" width="2" height="2"/><rect x="11" y="12" width="2" height="2"/><rect x="15" y="12" width="2" height="2"/><rect x="19" y="12" width="2" height="2"/><rect x="23" y="12" width="2" height="2"/><rect x="7" y="16" width="2" height="2"/><rect x="11" y="16" width="2" height="2"/><rect x="15" y="16" width="2" height="2"/><rect x="19" y="16" width="2" height="2"/><rect x="23" y="16" width="2" height="2"/><rect x="7" y="20" width="2" height="2"/><rect x="11" y="20" width="2" height="2"/><rect x="15" y="20" width="2" height="2"/><rect x="19" y="20" width="2" height="2"/><rect x="23" y="20" width="2" height="2"/><rect x="7" y="24" width="2" height="2"/><rect x="11" y="24" width="2" height="2"/><rect x="15" y="24" width="2" height="2"/><rect x="19" y="24" width="2" height="2"/><path d="M40,12H36V10a1,1,0,0,0-2,0v2H32V10a1,1,0,0,0-2,0v2H28V10a1,1,0,0,0-2,0v2H24V10a1,1,0,0,0-2,0v2H20V10a1,1,0,0,0-2,0v2H16V10a1,1,0,0,0-2,0v2H10a1,1,0,0,0-1,1V40a1,1,0,0,0,1,1H40a1,1,0,0,0,1-1V13A1,1,0,0,0,40,12ZM11,39V14h3v2a1,1,0,0,0,2,0V14h2v2a1,1,0,0,0,2,0V14h2v2a1,1,0,0,0,2,0V14h2v2a1,1,0,0,0,2,0V14h2v2a1,1,0,0,0,2,0V14h2v2a1,1,0,0,0,2,0V14h3V39Z" transform="translate(-9 -9)"/></svg>
				<p>割引上映中の劇場</p>
			</div>		
		</div>
	</div>
</div>
<div class="calendar">
	<ul class="list-group list-group-horizontal">
		<li v-bind:id="'calendar-' + [index]" v-bind:ref="[todo.id]" class = "list-group-item flex-fill" v-bind:class="{ enable: todo.touch,active: index === day_button_index }" v-for="(todo,index) in day_button_list" :key="todo.id" v-on:click="daybtnClick(index)"><p class="day">{{todo.num}}<span>{{todo.str}}</span></p></li>
	</ul>
</div>
</div>
</template>

<script>
import moment from 'moment';
moment.locale('ja');
import geolist from '../assets/geo_list.json';
import salesfilter from '../assets/sales_filter.json';

export default {
  data () {
    return {
	  date:moment().format("HH:mm"),
	  day_button_list:[],
	  menu_button_index:0,
	  day_button_index:0,
	  geo_list:geolist,
	  selected_area: null,
	  overlay:false,
	  selected_sales:null,
      sales_filter: salesfilter
    }
  },
  props: {
    area: Object,
	Data_calendar: Array,
	data_exist: Boolean
  },
  mounted () {
    	setInterval(()=>{
			this.date = moment().format("HH:mm");
		},1000);
		
		$(document).ready(function() {
			 $('.drawer').drawer({
				 showOverlay: false
			 });
		});
		if(this.$parent.response_data == null) {
			var calendar = [];
			for (var i = 0; i < 5; i++){
			  var id,day_num,day_ja,week,month;
			  var touch = false;
			  month = moment().add(i,'days').format('MM');
			  day_num = moment().add(i,'days').format('DD');
			  day_ja = moment().add(i,'days').format('dd');
			  id = month + day_num;
			  var num = month + "/" + day_num;
			  var str = "（" + day_ja + "）";	
			  calendar.push({id:id,num:num,str:str,filter:month + day_num,touch:touch});
		    }   
			this.day_button_list = calendar;
			this.day_button_index = null;
		}
  },
  methods: {
    daybtnClick:function(index){
		let id = this.day_button_list[index].id;
		this.$parent.time["day"] = moment().add(index, 'days').date();
		this.$parent.time["week"] = moment().add(index, 'days').weekday();
		this.$parent.time["day_select"] = id;
		this.day_button_index = index;
		this.$emit('callParent_refresh');
    },
    menubtnClick:function(index){
		if(index != this.menu_button_index) {
			let category = {
				0:"theater",
				1:"titles",
				2:"sales"
			};
			this.menu_button_index = index;
			this.$parent.category = category[index];
			this.$parent.Data_calendar = [];
			this.$emit('callParent_refresh');
		}
    },
    reloadClick:function(category){
		this.$parent.loading = true;
		this.overlay = false;
		this.$parent.overlay = false;
		switch (category){
			case 0:
				this.menu_button_index = 0;
				this.day_button_index = 0;
				this.$parent.category = "theater";
				this.$parent.time["day"] = moment().add(0, 'days').date();
				this.$parent.time["week"] = moment().add(0, 'days').weekday();
				this.$parent.time["day_select"] = this.day_button_list[0].id;
				this.$parent.filter_num = null;
				this.$emit('callParent_reload');
				this.$parent.area.pref = null;
				this.$parent.area.city = null;
				this.selected_area = null;
				break;
			case 1:
				this.menu_button_index = 0;
				this.day_button_index = 0;
				this.$parent.category = "theater";
				this.$parent.time["day"] = moment().add(0, 'days').date();
				this.$parent.time["week"] = moment().add(0, 'days').weekday();
				this.$parent.time["day_select"] = this.day_button_list[0].id;
				this.$parent.filter_num = null;	
				this.$emit('callParent_rearea');
				break;
			case 2:
				this.$parent.loading = false;
				this.$emit('callParent_refresh');
				break;
		}
		$('.drawer').drawer('close');
    },
    setting:function(){
		if(!this.overlay) {
			console.log("A");
			this.overlay = true;
			this.$parent.overlay = true;
		} else {
			console.log("B");
			this.overlay = false;
			this.$parent.overlay = false;
		}
    }
  },
    watch: {
	  data_exist: function (val, oldVal) {
		var calendar = [];
		for (var i = 0; i < 5; i++){
			var id,day_num,day_ja,week,month;
			var touch = false;
			month = moment().add(i,'days').format('MM');
			day_num = moment().add(i,'days').format('DD');
			day_ja = moment().add(i,'days').format('dd');
			id = month + day_num;
			var num = month + "/" + day_num;
			var str = "（" + day_ja + "）";	
			calendar.push({id:id,num:num,str:str,filter:month + day_num,touch:touch});
		 }   
		 this.day_button_list = calendar;
		 this.day_button_index = null;
		 this.menu_button_index = null;
	  },
      Data_calendar: function (val, oldVal) {
		  var calendar = [];
		  var today = moment().format('d');
		    
		  for (var i = 0; i < 5; i++){
			  var id,day_num,day_ja,week,month;
			  var touch = false;
			  month = moment().add(i,'days').format('MM');
			  day_num = moment().add(i,'days').format('DD');
			  day_ja = moment().add(i,'days').format('dd');
			  id = month + day_num;
			
			  var num = month + "/" + day_num;
			  var str = "（" + day_ja + "）";	
			  week = moment().add(i,'days').format('d');
			  if (this.Data_calendar.indexOf(id) >= 0){
				  touch = true;
			  }
			  calendar.push({id:id,num:num,str:str,filter:month + day_num,touch:touch});
		  }
		   
		  this.day_button_list = calendar;
		  
		  this.day_button_index = 0;
		  
		  console.log(this.day_button_list);
		  
		  if (!this.day_button_list[0].touch) {
			  console.log("null");
			  this.day_button_index = null;
		  } else {
			  this.daybtnClick(0);
		  }
		  
		  if( this.day_button_index == null) {
			  for (var i = 0; i < this.day_button_list.length; i++) { 
				  if(this.day_button_list[i].touch) {
					  this.day_button_index = i;
					  this.daybtnClick(i);
					  break;
				  }
			  }	  
		  }
		  
	  },
      selected_area: function (val, oldVal) {
		if(val !== null) {
			this.$parent.geo["lat"] = val.lat;
			this.$parent.geo["lng"] = val.lng;
			this.$parent.gps = false;
			this.$parent.area.pref = val.txt;
			this.$parent.area.city = null;
			this.$parent.Data_calendar = [];
			this.reloadClick(1);
		}
	  },
      selected_sales: function (val, oldVal) {
		 this.$parent.sales_filter = this.selected_sales;
		 this.$parent.Data_calendar = [];
		 this.reloadClick(2);
	  } 
    }
}



</script>