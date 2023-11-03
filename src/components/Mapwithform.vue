<!-- MapWithForm.vue -->
<template>
    <div>
        <form @submit.prevent="submitForm" class="form-grid">
            <div class="form-group">
                <label for="cep">CEP:</label>
                <input id="cep" type="text" v-model="cep" required />
            </div>
            <div class="form-group">
                <label for="endereco">Endereço:</label>
                <input id="endereco" type="text" v-model="endereco" />
            </div>
            <div class="form-group">
                <label for="numero">Número:</label>
                <input id="numero" type="text" v-model="numero" />
            </div>
            <div class="form-group">
                <label for="bairro">Bairro:</label>
                <input id="bairro" type="text" v-model="bairro" />
            </div>
            <div class="form-group">
                <label for="complemento">Complemento:</label>
                <input id="complemento" type="text" v-model="complemento" />
            </div>
            <div class="form-group">
                <label for="estado">Estado:</label>
                <input id="estado" type="text" v-model="estado" />
            </div>
            <div class="form-group">
                <label for="municipio">Município:</label>
                <input id="municipio" type="text" v-model="municipio" />
            </div>
            <button type="submit">Buscar Localização</button>
        </form>
        <button @click="clearMarker" class="clear-button">
            <img src="path-to-trash-icon.png" alt="Apagar georrefenciamento" />
        </button>
        <button @click="startDrawing" class="clear-button">
            <img src="path-to-pencil-icon.png" alt="Desenhar" />
        </button>
        <button @click="clearDraw" class="clear-button">
            <img src="path-to-trash-icon.png" alt="Limpar contexto temporário" />
        </button>
        <button @click="measureDistance" class="clear-button">
            <img src="path-to-trash-icon.png" alt="Medir Distancia" />
        </button>
        <button @click="addMarkerOnClick" class="clear-button">
            Georreferenciar
        </button>
        <button @click="buscarRua" class="clear-button">
            Localizar endereço
        </button>

        <div ref="map" class="map"></div>
    </div>
</template>
  
<script>
import 'ol/ol.css';
import { Map, View, Feature } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import { Point } from 'ol/geom';
import { Style, Icon } from 'ol/style';
import { Vector as VectorSource } from 'ol/source';
import VectorLayer from 'ol/layer/Vector';
import Draw from 'ol/interaction/Draw.js';


export default {
    name: 'MapWithForm',
    data() {
        return {
            cep: '',
            endereco: '',
            numero: '',
            bairro: '',
            complemento: '',
            estado: '',
            municipio: '',
            isAddMarkerButtonClicked: false,
            map: null,
            draw: null,
            vectorLayer: new VectorLayer({
                source: new VectorSource({ wrapX: false }),
            }),
            markerLayer: new VectorLayer({
                source: new VectorSource(),
            }),
            uniqueFeatures: new Set(),
        };
    },
    mounted() {
        this.setupMap();
    },
    methods: {
        setupMap() {
            const source = new VectorSource({ wrapX: false });
            const se = new VectorLayer({
                source: source,
            });
            const raster = new TileLayer({
                source: new OSM(),
            });
            this.map = new Map({
                target: this.$refs.map,
                layers: [raster, se,
                    new TileLayer({
                        source: new OSM(),
                    }),
                ],
                view: new View({
                    center: [-5429488.227348947, -3198201.0568349618],
                    zoom: 8,
                }),
            });

            this.map.on('click', this.handleMapClick)
        },

        addMarkerOnClick() {
            this.isAddMarkerButtonClicked = true;
        },

        clearDraw() {
            if (this.draw) {
                this.map.removeInteraction(this.draw);
                this.vectorLayer.getSource().clear();
                this.map.removeLayer(this.vectorLayer);
                this.draw = null;


                this.uniqueFeatures.forEach(feature => {
                    this.vectorLayer.getSource().removeFeature(feature);
                });


                this.uniqueFeatures.clear();
            }
        },

        clearMarker() {
            if (this.markerLayer) {
                this.map.removeLayer(this.markerLayer);
                this.markerLayer = null;
                this.markerFeature = null;
            }
        },


        startDrawing() {
            if (this.draw) {
                this.map.removeInteraction(this.draw);
            }

            const source = new VectorSource({ wrapX: false });

            this.draw = new Draw({
                source: source,
                type: 'Polygon',
            });

            this.draw.on('drawend', (event) => {
                const feature = event.feature;

                if (!this.uniqueFeatures.has(feature)) {
                    this.vectorLayer.getSource().addFeature(feature);
                    this.uniqueFeatures.add(feature);
                }
            });

            this.map.addLayer(this.vectorLayer);
            this.map.addInteraction(this.draw);
        },



        handleMapClick(event) {
            const coordinates = event.coordinate;
            console.log('Clicked at Coordinates:', coordinates);
            this.addMarker(coordinates);
        },

        addMarker(coordinates) {
            if (this.isAddMarkerButtonClicked) {
                if (this.markerFeature) {
                    this.markerFeature.getGeometry().setCoordinates(coordinates);
                } else {
                    this.markerFeature = new Feature({
                        geometry: new Point(coordinates),
                    });

                    const markerStyle = new Style({
                        image: new Icon({
                            anchor: [0.5, 1],
                            src: '../../img/icon.png',
                        }),
                    });

                    this.markerFeature.setStyle(markerStyle);

                    this.markerLayer = new VectorLayer({
                        source: new VectorSource({
                            features: [this.markerFeature],
                        }),
                    });

                    this.map.addLayer(this.markerLayer);
                }
                this.isAddMarkerButtonClicked = false;
            }
        },

        measureDistance() {
            if (this.draw) {
                this.map.removeInteraction(this.draw);
            }

            const source = new VectorSource({ wrapX: false });

            this.draw = new Draw({
                source: source,
                type: 'LineString',
            });

            this.draw.on('drawend', (event) => {
                const feature = event.feature;
                this.vectorLayer.getSource().addFeature(feature);
                const geometry = feature.getGeometry();
                const coordinates = geometry.getCoordinates();
                const length = Math.round(geometry.getLength() * 100) / 100;
                console.log(`Distância medida: ${length} metros`);
            });
            this.map.addLayer(this.vectorLayer);
            this.map.addInteraction(this.draw);
        },

        async buscarRua() {
            try {
                const resposta = await fetch(`https://nominatim.openstreetmap.org/search?q=${this.endereco}&format=json`);
                const dados = await resposta.json();

                if (dados && dados.length > 0) {
                    const firstResult = dados[0];
                    const latitude = parseFloat(firstResult.lat);
                    const longitude = parseFloat(firstResult.lon);

                    const coordinates = fromLonLat([longitude, latitude]);
                    this.map.getView().setCenter(coordinates);
                    this.map.getView().setZoom(17);


                    this.clearDraw();

                    this.clearMarker();
                    this.addMarker(coordinates);

                } else {
                    console.error('Endereço não encontrado.');
                }
            } catch (erro) {
                console.error(erro);
            }
        },



        async submitForm() {
            try {
                if (this.cep) {
                    const response = await fetch(`https://viacep.com.br/ws/${encodeURIComponent(this.cep)}/json/`);
                    const data = await response.json();

                    if (!data.erro) {
                        this.endereco = data.logradouro || '';
                        this.numero = data.numero || '';
                        this.bairro = data.bairro || '';
                        this.complemento = data.complemento || '';
                        this.estado = data.uf || '';
                        this.municipio = data.localidade || '';

                        console.log('Formulário enviado:', {
                            cep: this.cep,
                            endereco: this.endereco,
                            numero: this.numero,
                            bairro: this.bairro,
                            complemento: this.complemento,
                            estado: this.estado,
                            municipio: this.municipio,
                        });

                        this.updateMap();
                    } else {
                        console.error('CEP não encontrado.');
                    }
                } else {
                    console.error('Campo "CEP" é obrigatório.');
                }
            } catch (error) {
                console.error('Erro ao buscar localização:', error.message);
            }
        },
        async updateMap() {
            try {
                const response = await fetch(`https://brasilapi.com.br/api/cep/v2/${this.cep.replace("/-|\./gi, ''")}`);
                const data = await response.json();
                console.log(data);

                if (data) {
                    const result = data;
                    const coordinates = fromLonLat([parseFloat(result.location.coordinates.longitude), parseFloat(result.location.coordinates.latitude)]);
                    this.map.getView().setCenter(coordinates);
                    this.map.getView().setZoom(10);

                    const [longitude, latitude] = coordinates.map(coord => coord.toFixed(6));

                    console.log(`Latitude: ${latitude}, Longitude: ${longitude}`);
                } else {
                    console.error('Localização não encontrada para o CEP fornecido.');
                }
            } catch (error) {
                console.error('Erro ao buscar localização:', error.message);
            }
        },






    },
    watch: {
        vectorLayerFeatures: {
            handler(newFeatures) {
                this.vectorLayer.getSource().clear();
                this.vectorLayer.getSource().addFeatures(newFeatures);
            },
            deep: true,
        },
    },
};
</script>
  
<style scoped>
.form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
}

.form-group {
    margin-bottom: 15px;
}

.map {
    width: 100%;
    height: 400px;
}

.clear-button img {
    width: 20px;
}
</style>
  