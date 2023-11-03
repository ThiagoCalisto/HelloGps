<!-- MapWithForm.vue -->
<template>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
    <div class="container">
        <form @submit.prevent="submitForm">
            <div class="group-one">
                <div class="form-group">
                    <div class="input-with-icon">
                        <label for="cep">CEP:</label>
                        <div class="input-with-icon-div">
                            <input id="cep" type="text" v-model="cep" required />
                            <button class="search-button" type="submit">
                                <img class="search-icon" src="../../../img/icons8-search-50.png" alt="Ícone de pesquisa">
                            </button>
                        </div>
                    </div>
                </div>

                <div class="form-group-secondary" style="flex: 2;">
                    <label for="endereco">Endereço:</label>
                    <input id="endereco" type="text" v-model="endereco" />
                </div>
                <div class="form-group-secondary">
                    <label for="numero">Número:</label>
                    <input id="numero" type="text" v-model="numero" />
                </div>
                <div class="form-group-secondary">
                    <label for="bairro">Bairro:</label>
                    <input id="bairro" type="text" v-model="bairro" />
                </div>
            </div>

            <div class="group-one">
                <div class="form-group-secondary" style="flex: 2;">
                    <label for="complemento">Complemento:</label>
                    <input id="complemento" type="text" v-model="complemento" />
                </div>
                <div class="form-group-secondary">
                    <label for="estado">Estado:</label>
                    <select id="estado" v-model="estado">
                        <option value="DF">Distrito Federal</option>
                        <option value="GO">Goiás</option>
                        <option value="MG">Minas Gerais</option>
                    </select>
                </div>
                <div class="form-group-secondary">
                    <label for="municipio">Município:</label>
                    <select id="municipio" v-model="municipio">
                        <option value="Brasília">Brasília</option>
                        <option value="Goiânia">Goiânia</option>
                        <option value="Belo Horizonte">Belo Horizonte</option>
                    </select>
                </div>
            </div>
        </form>

        <div class="map-container">
            <div ref="map" class="map">
            </div>
            <div class="map-buttons">
                <button @click="addMarkerOnClick" type="submit">
                    <img src="../../img/pointer.png" style="width:23px; height:23px;">
                </button>
                <button @click="startDrawing" type="submit">
                    <img src="../../img/regua-triangular.png" style="width:23px; height:23px;">
                </button>
                <button @click="measureDistance" type="submit">
                    <img src="../../img/governante.png" style="width:23px; height:23px;">
                </button>
                <button @click="clearDraw" type="submit">
                    <img src="../../img/borracha.png" style="width:23px; height:23px;">
                </button>
                <button @click="buscarRua" type="submit">
                    <img src="../../img/mapa.png" style="width:23px; height:23px;">
                </button>
                <button @click="clearMarker" type="submit">
                    <img src="../../img/lixo.png" style="width:23px; height:23px;">
                </button>
            </div>
        </div>
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
import { Text } from 'vue';
import { LineString } from 'ol/geom';
import { getLength } from 'ol/sphere';



export default {
    name: "MapWithForm",
    data() {
        return {
            cep: "",
            endereco: "",
            numero: "",
            bairro: "",
            complemento: "",
            estado: "",
            municipio: "",
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

            const projection = new View({
                center: [-48.5480, -27.5954],
                zoom: 10,
                projection: 'EPSG:4326',
            });

            this.map = new Map({
                target: this.$refs.map,
                layers: [raster, se, new TileLayer({
                    source: new OSM(),
                }),
                ],
                view: projection,
            });

            this.map.on("click", this.handleMapClick);
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
                type: "Polygon",
            });
            this.draw.on("drawend", (event) => {
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
            console.log("Clicked longitude:", coordinates[0]);
            console.log("Clicked latitude:", coordinates[1]);
            this.addMarker(coordinates);
        },
        addMarker(coordinates) {
            if (this.isAddMarkerButtonClicked) {
                if (this.markerFeature) {
                    this.markerFeature.getGeometry().setCoordinates(coordinates);
                }
                else {
                    this.markerFeature = new Feature({
                        geometry: new Point(coordinates),
                    });
                    const markerStyle = new Style({
                        image: new Icon({
                            anchor: [0.5, 1],
                            src: "../../img/placeholder.png",
                            scale: 0.1,
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
                type: "LineString",
            });

            this.draw.on("drawend", (event) => {
                const feature = event.feature;
                this.vectorLayer.getSource().addFeature(feature);

                const geometry = feature.getGeometry();
                const coordinates = geometry.getCoordinates();

                
                const transformedCoordinates = coordinates.map(coord =>
                    fromLonLat(coord, 'EPSG:4326')
                );

                const lineString = new LineString(transformedCoordinates);
    
                const wgs84Length = getLength(lineString, { projection: 'EPSG:4326' });

                const formattedLength = wgs84Length.toFixed(2);

                console.log(`Distância medida: ${formattedLength} metros no sistema WGS84`);
            });

            this.map.addLayer(this.vectorLayer);
            this.map.addInteraction(this.draw);
        },

        async buscarRua() {
            try {
                const response = await fetch(`https://nominatim.openstreetmap.org/search?q=${this.endereco}&format=json`);
                const dados = await response.json();
                console.log(dados);

                if (dados && dados.length > 0) {
                    const firstResult = dados[0];
                    const latitude = parseFloat(firstResult.lat);
                    const longitude = parseFloat(firstResult.lon);

                    // Ajuste para coordenadas no formato EPSG:4326 (WGS84)
                    const coordinates = [longitude, latitude];

                    this.map.getView().setCenter(coordinates);
                    this.map.getView().setZoom(17);
                    this.clearDraw();
                    this.clearMarker();
                    this.addMarker(coordinates);
                } else {
                    console.error("Endereço não encontrado.");
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
                        this.endereco = data.logradouro || "";
                        this.numero = data.numero || "";
                        this.bairro = data.bairro || "";
                        this.complemento = data.complemento || "";
                        this.estado = data.uf || "";
                        this.municipio = data.localidade || "";
                        console.log("Formulário enviado:", {
                            cep: this.cep,
                            endereco: this.endereco,
                            numero: this.numero,
                            bairro: this.bairro,
                            complemento: this.complemento,
                            estado: this.estado,
                            municipio: this.municipio,
                        });
                        this.updateMap();
                    }
                    else {
                        console.error("CEP não encontrado.");
                    }
                }
                else {
                    console.error("Campo \"CEP\" é obrigatório.");
                }
            }
            catch (error) {
                console.error("Erro ao buscar localização:", error.message);
            }
        },
        async updateMap() {
            try {
                const response = await fetch(`https://brasilapi.com.br/api/cep/v2/${this.cep.replace(/-|\./gi, '')}`);
                const data = await response.json();

                console.log(data);

                if (data) {
                    const result = data;

                    const coordinates = [parseFloat(result.location.coordinates.longitude), parseFloat(result.location.coordinates.latitude)];

                    this.map.getView().setCenter(coordinates);
                    this.map.getView().setZoom(10);
                    const [longitude, latitude] = coordinates.map(coord => coord.toFixed(6));
                    console.log(`Latitude: ${latitude}, Longitude: ${longitude}`);
                } else {
                    console.error("Localização não encontrada para o CEP fornecido.");
                }
            } catch (error) {
                console.error("Erro ao buscar localização:", error.message);
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
    components: { Text }
};
</script>
  
<style scoped>
.container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.group-one {
    display: flex;
    flex-wrap: wrap;
    margin-bottom: 10px;
    gap: 20px;
}

.form-group label {
    flex: 1;
    min-width: 100px;
    text-align: right;
}

.form-group input {
    flex: 2;
    width: 100%;
    height: 30px;
}

.input-with-icon {
    text-align: start;
    flex-direction: column;
}

.input-with-icon-div {
    display: flex;
    align-items: center;
}

.search-button {
    background-color: #00bcd4;
}

.search-icon {
    width: 22px;
    height: 22px;
    filter: invert(100%)
}

.form-group-secondary {
    display: flex;
    flex-direction: column;
    text-align: start;
}

.form-group-secondary input {
    width: 100%;
    height: 30px;
}

.form-group-secondary select {
    width: 100%;
    height: 30px;
}

.map {
    width: 100%;
    height: 400px;
}

.map-container {
    position: relative;
}

.map-buttons {
    position: absolute;
    top: 10px;
    right: 10px;
    display: inline-block;
    flex-direction: column;
}

.clear-button img {
    width: 20px;
}
</style>
  