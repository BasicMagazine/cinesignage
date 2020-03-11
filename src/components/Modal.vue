<template>
<div>
  <b-modal ok-only hide-header-close no-close-on-backdrop no-close-on-esc
   id="modal-1" title="都道府県を選択" v-model="modalShow" @ok="handleOK">
  <p>現在地を取得できませんでした。</p>
  <p>最寄りの都道府県を選択してください。</p>
  <b-form-select v-model="selected" :options="geo_list" required v-bind:class="{ caution: caution }"></b-form-select>
  <span v-show="caution">都道府県が選択されていません</span>
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
			caution:false,
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
				bvModalEvt.preventDefault();
				this.caution = true;
			} else {
			this.caution = false;
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