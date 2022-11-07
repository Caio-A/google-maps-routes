<template>
    <div>
        <section class="ui one column centered grid" style="position:relative;z-index:1;">
            <div class="column"></div>
            <form class="ui segment large form">
                <div class="ui message blue" v-show="error">{{error}}</div>
                <div class="ui segment">
                    <div class="field">
                        <div class="ui right icon input large" :class="{loading:spinner}">
                            <input id="autocomplete" type="text" v-model="address" placeholder="Partida">
                            <input id="destination" type ="text" v-model="destination" placeholder="Destino">
                            <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                        </div>
                    </div>
                    <button class="ui button" @click="getLatLngPD">Go!</button>
                </div>
            </form>
        </section>
        <section id="map">
        </section>
    </div>
</template>


<script>

import axios from 'axios'

export default {
    data() {
        return {
            address: "",
            error: "",
            spinner: false,
            destination : "",
        }
    },
    mounted(){
        //Autocomplete do destino
        let destination = new google.maps.places.Autocomplete(
            document.getElementById("destination"),{
                bounds : new google.maps.LatLngBounds(
                    new google.maps.LatLng(-21.7883, -46.5625)
                )
            }
        );
        destination.addListener("place_changed", () => {
            let place = destination.getPlace();
            console.log(place);
        })

        //Autocomplete do ponto de partida
        let autocomplete = new google.maps.places.Autocomplete(
            document.getElementById("autocomplete"),
            {
                bounds : new google.maps.LatLngBounds(
                    new google.maps.LatLng(-21.7883, -46.5625)
                )
            }
        );

        autocomplete.addListener("place_changed", () => {
            let place = autocomplete.getPlace();
            console.log(place);
            this.showUserLocationOnTheMap(
                place.geometry.location.lat(), 
                place.geometry.location.lng()
            )
        });
    },
    methods: {
        locatorButtonPressed(){
            this.spinner = true;
            if(navigator.geolocation){
                navigator.geolocation.getCurrentPosition(position => {
                    this.getAddressFrom(
                        position.coords.latitude, 
                        position.coords.longitude); 
                    this.showUserLocationOnTheMap(
                        position.coords.latitude, 
                        position.coords.longitude);
                },
                error => {
                    this.spinner = false;
                    this.error = error.message;
                    console.log(error.message);
                }
                );
            } else {
                this.error = error.message;
                this.spinner = false;
                console.log("Your browser does not support geolocation API");

            }
        },
        getAddressFrom(lat, long) {
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" + lat + "," + long + "&key=AIzaSyDoZfCPN_l1TYd8lqmLzwmG9vn74WXC3WM")
            .then(response =>{
                if(response.data.error_message){
                    this.error = response.data.error_message;
                    console.log(response.data.error_message);
                }
                else{
                    this.address = response.data.results[0].formatted_address;
                    console.log(response.data.results[0].formatted_address);
                }
                this.spinner = false;
            })
            .catch(error => {
                this.error = error.message;
                this.spinner = false;
                console.log(error.message);
            })
        },
        showUserLocationOnTheMap(latitude, longitude){
            let map = new google.maps.Map(document.getElementById("map"), {
                zoom: 15,
                center: new google.maps.LatLng(latitude, longitude),
                mapTypeId : google.maps.MapTypeId.ROADMAP,
            });
            let start = new google.maps.Marker({
                position : new google.maps.LatLng(latitude, longitude),
                map:map
            })
        },
        makeRoteOnTheMap(latitudeP, longitudeP, latitudeD, longitudeD){
            var heatMapData = [
                new google.maps.LatLng(latitudeP, longitudeP),
                new google.maps.LatLng(latitudeD, longitudeD),
        ]
            const directionsService = new google.maps.DirectionsService();
            const directionsRenderer = new google.maps.DirectionsRenderer({
                draggable : true,
            });

            let partida = new google.maps.LatLng(latitudeP, longitudeP);
            let destino = new google.maps.LatLng(latitudeD, longitudeD);

            let map = new google.maps.Map(document.getElementById("map"), {
                zoom : 15,
                center : new google.maps.LatLng(latitudeP,longitudeP),
                mapTypeId : google.maps.MapTypeId.ROADMAP,
            });

            let start = new google.maps.Marker({
                position : new google.maps.LatLng(latitudeP, longitudeP),
                map: map
            });
            let dest = new google.maps.Marker({
                position : new google.maps.LatLng(latitudeD, longitudeD),
                map: map
            });

            var heatmap = new google.maps.visualization.HeatmapLayer({
                data : heatMapData,
            });

            heatmap.setMap(map);

            directionsRenderer.setMap(map);

            directionsService.route({
                origin: partida,
                destination: destino,
                travelMode : google.maps.TravelMode.DRIVING
            }).then(response => {
                console.log({response});
                directionsRenderer.setDirections(response);
            }).catch(error => {
                this.error = error.message;
            })

            
        },
        getLatLngPD(){
            let destination = document.getElementById("destination").value;
            let start = document.getElementById("autocomplete").value;
            start = start.replaceAll(' ', '%20');
            destination = destination.replaceAll(' ', '%20');
            console.log(destination);
            console.log(start);
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + start +  "&key=AIzaSyDoZfCPN_l1TYd8lqmLzwmG9vn74WXC3WM")
            .then(response =>{
                var latitudeP = response.data.results[0].geometry.location.lat;
                var longitudeP = response.data.results[0].geometry.location.lng;
            
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + destination +  "&key=AIzaSyDoZfCPN_l1TYd8lqmLzwmG9vn74WXC3WM")
            .then(response =>{
                var latitudeD = response.data.results[0].geometry.location.lat;
                let longitudeD = response.data.results[0].geometry.location.lng;

                console.log(latitudeP);
                console.log(longitudeP);
                console.log(latitudeD);
                console.log(longitudeD);
                this.makeRoteOnTheMap(latitudeP, longitudeP, latitudeD, longitudeD);
            });
            });
        }
    }
}
</script>

<style>
.ui.button,
.dot.circle.icon{
    background-color: rgb(58, 104, 255);
    color: white;
}
.pac-icon{
    display: none;
}
.pac-item{
    padding: 10px;
    font-size: 16px;
    cursor: pointer;
}
pac-item:hover{
    background-color: #ececec; 
}

.pac-item-query {
    font-size: 16px;
}

#map {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background:rgb(58, 104, 255)
}
</style>
