<template>
<div class="area_button_space">
	<button type="button" v-bind:class="{ active: index === area_button_index }" v-for="(area,index) in Footer_list" :key="area.id" v-on:click="areabtnClick(area,index)">{{area}}</button>
</div>
</template>

<script>

var touch = true;

export default {
  data () {
    return {
	  area_button_index:0
    }
  },
  props: {
	Footer_list: Array,
	filter_num_parent: Number
  },
  mounted () {
  },
  methods: {
    areabtnClick:function(area,index){
		this.area_button_index = index;
		if(index == 0) {
			this.$parent.filter_num = null;
			this.$parent.area.select_area = null;
			if(this.$parent.gps) {
				this.$parent.area.location_txt = this.$parent.area.current_location;
			} else {
				this.$parent.area.location_txt = this.$parent.area.pref;
			}
		} else {
			this.$parent.filter_num = index - 1;
			this.$parent.area.select_area = area;
			this.$parent.area.location_txt = this.$parent.area.pref + this.$parent.area.select_area;
		}
		this.$parent.Data_calendar = [];
		this.$emit('callParent_refresh');
    }
  },
  watch: {
      filter_num_parent: function (val, oldVal) {
		 if(val == null) {
			 this.area_button_index = 0;
		 }
	  }
  }
}



</script>