<template>
<div>
  
  <b-modal ok-only hide-header-close no-close-on-backdrop no-close-on-esc
   id="modal-1" title="都道府県を選択" v-model="modalShow" @ok="handleOK">
  <p>現在地を取得できませんでした。</p>
  <p>最寄りの都道府県を選択してください。</p>
  <b-form-select v-model="selected" :options="geo_list"></b-form-select>
  </b-modal>


</div>
  
</div>
</template>

<script>

import geolist from '../assets/geo_list.json'

export default {
	data () {
		return {
			modalShow: false,
			selected: null,
			geo_list:geolist
		}
	},
	props: {
		modalView: Boolean
	},
	methods: {
		handleOK(bvModalEvt) {
			console.log(this.selected);
			if (this.selected == null) {
				console.log("test");
				bvModalEvt.preventDefault()
			} else {
			this.$parent.geo["lat"] = this.selected.lat;
			this.$parent.geo["lng"] = this.selected.lng;
			this.$parent.gps = false;
			this.$parent.area.pref = this.selected.txt;
			this.$parent.area.city = null;
			this.$emit('callParent_rearea');
			this.$parent.loading = true;				
			}
		
		

		}
	},
    watch: {
	  modalView: function (val, oldVal) {
		this.modalShow = val;
	  }
    }
}
</script>