<template>
	<div class="map" id="map-container"></div>
</template>

<script>

	import {eventBus} from '../../main'
	import {updateMapLayer, prettifyMapLayer} from '../../util/styleSync'

	import mapboxgl from 'mapbox-gl/dist/mapbox-gl.js'
	import GlSpec from '@mapbox/mapbox-gl-style-spec/reference/latest'
	import validateStyleMin from '@mapbox/mapbox-gl-style-spec/validate_style.min'

	import * as types from '../../store/mutation-types'
	import {mapState, mapMutations, mapGetters} from 'vuex'
	import {forOwn, get, keyBy, reduce} from 'lodash'

	mapboxgl.accessToken = 'pk.eyJ1IjoicGx1Z24iLCJhIjoiY2l6cHIyejhzMDAyODJxdXEzaHM2cmVrZiJ9.qLg-Ki18d0JQnAMfzg7nCA';

	let map,
		_initialStyle;

	export default {
		data() {
			return {map:map};
		},
		computed:{
			...mapState([
				'vStyle',
				'smartUpdate'
			]),

			...mapGetters([
				'getPopupFeatures',
				'getLayerIndex'
			])
		},

		created() {
			eventBus.$on('map:resize', function () {
				console.log('@map:resize');
				if (map) {
					map.resize();
				}
			});

			this.$watch('vStyle', this.track_vStyle, {deep: true});
		},

		mounted() {
			this.initMap();
			eventBus.$on('map:layer.update', this.onLayerUpdate);
		},

		methods: {
			...mapMutations({
				setMapPopup: types.SET_MAP_POPUP,
				setLayer: types.SET_LAYER,
				setSmartUpdate: types.SET_SMART_UPDATE
			}),

			track_vStyle(vStyle) {
				if (!vStyle) { return; }

				if (!map) {
					_initialStyle = vStyle;
					return;
				}

				if (!this.smartUpdate.length) {
					this.applyStyle(vStyle);
					return;
				}

				forOwn(this.smartUpdate, function (cmdItem) {
					let [mapMethod, params] = cmdItem;
					map[mapMethod].apply(map, params);
					console.log(' # smartUpdate : ', mapMethod+'( ' + params.join(', ') + ' )');
				});
				this.setSmartUpdate([]);


			},

			onLayerUpdate(layerId, values, layerNewStyle) {
				console.log('map:layer.update', layerId, values, layerNewStyle);

				let options = {value: layerNewStyle, key: '', style: this.vStyle, styleSpec: GlSpec }  ;
				let errors = validateStyleMin.layer(options);
				console.warn(' * onLayerUpdate() errors : ', _.map(errors, 'message'));
				if (errors.length) {return;}

				let {cmd, unhandledParams} = updateMapLayer(layerId, values, map);

				if (!Object.keys(unhandledParams).length) {
					this.setSmartUpdate(cmd);
				}

				this.setLayer({layerId: layerId, value: layerNewStyle});
			},

			applyStyle(value) {
				map.setStyle(value);
			},

			initMap() {
				const vm = this;
				map = new mapboxgl.Map({
					container: this.$el,
					center: [37, 55],
					zoom: 5,
					hash: true // display current zoom and coordinates as a part of URL
				});

				if (_initialStyle) {
				    this.applyStyle(_initialStyle);
				    _initialStyle = undefined;
				}

				let nav = new mapboxgl.NavigationControl();
				map.addControl(nav, 'top-left');

				function onInteraction(e) {
					if (vm.getPopupFeatures && vm.getPopupFeatures.length) {
						vm.setMapPopup({features:[]});
					}
				}

				'dragstart zoomstart resize'.split(' ').forEach(function(event){
					map.on(event, onInteraction);
				})

				map.on('click', function (e) {
					if (vm.getPopupFeatures && vm.getPopupFeatures.length) {
						return onInteraction(e);
					}
					let mapFeatures = map.queryRenderedFeatures(e.point, {});
					let oFeatures = keyBy(mapFeatures, 'layer.id');
					let features = reduce(oFeatures, (acc,v) => acc.concat(v), []);
					vm.setMapPopup({features, point: e.point});
				});
			}
		}
	}

</script>
<style src="mapbox-gl/dist/mapbox-gl.css"></style>

<style>
	.map {
		height: 100%;
		background-color: ghostwhite;
	}
	/*

	#map-container canvas {
		cursor: auto;
	}

	*/
</style>
