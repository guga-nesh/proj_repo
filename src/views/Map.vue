<template>
<header class="gradientBackground py-5">
    <div class="container pb-5">
        <div class="row gx-5 justify-content-center">
            <div class="col-lg-6">
                <div class="text-center">
                    <h1 class="main-title fw-bolder text-white">Map is the NEW way</h1>
                    <!--<p class="text-black subs"> Use our friendly maps to guide you to find the nearest hospitals/clinics </p>-->
                </div>
            </div>
        </div>
        <p class="text-black subs"> Use our friendly maps to guide you to find the nearest hospitals/clinics </p>
    </div>
</header>

<!-- MAP Features section-->
<section class="py-5 border-bottom" id="features">
    <div class="container px-5 my-5">
        <div class="row gx-5">
            <div class="col-lg-4 mb-5 mb-lg-0">
                <div class="form-header">
                    <h3>Search a location</h3>
                </div>
                    <p>Key in a location and find the nearest medical centres around it by clicking on "Find"</p>
                    <hr>
                    <h3>Want to find the nearest medical centres around you?</h3>
                    <p>Click the locator icon and click on "Find"</p>
                <div class="form-header" style="color:red;" v-show="error">
                    <h2>{{error}}</h2>
                </div>
                <div class="input-group">
                    <input
                        type="text"
                        id="autocomplete"
                        class="form-control"
                        placeholder="Enter a location"
                        v-model="address"
                    />
                    <button type="button" @click="userLocationButton" class="btn btn-outline-danger">
                        <i class="bi bi-geo-alt-fill"></i>
                    </button>
                </div>
                <br>
                <div>
                    <button @click="initialize" class="btn btn-outline-danger">Find</button>
                </div>
            </div>
                    
            <div class="col-lg-8 mb-5 mb-lg-0">
                <div id="map" style="height: 400px"></div>
            </div>
        </div>
    </div>
</section>

<!-- Team members section-->
<section class="gradientBackground py-5">
    <div class="container-fluid px-5 my-4 px-5">
        <div class="text-center mb-5">
            <h2 class="second-main fw-bolder text-white">Meet the Web Developers</h2>
            
            <p class="lead mb-0">"We're glad to make a change in our society"</p>
        </div>

        <!-- TEAM MEMBERS PHOTOS  -->
        <div class="container-fluid">
            <div class="teaminfo row">
                <div class="col-md-4 col-sm-12 col-12 indiv">
                    <img class="teamImg" src="../assets/teammates/guganesh.png" alt= "guganesh">
                    <p > Guganesh </p>
                    <p> Full Stack Developer </p>
                </div>

                <div class="col-md-4 col-sm-12 col-12 indiv">
                    <img class="teamImg" src="../assets/teammates/liqing.png" alt= "liqing">
                    <p> Li Qing </p>
                    <p> Full Stack Developer </p>
                </div>

                <div class="col-md-4 col-sm-12 col-12 indiv">
                    <img class="teamImg" src="../assets/teammates/avigale.png" alt= "avigale">
                    <p> Avigale </p>
                    <p> Full Stack Developer </p>
                </div>
            </div>
        <br/>

            <div class="teaminfo row">
                <div class="col-md-6 col-sm-12 col-12 indiv">
                    <img class="teamImg" src="../assets/teammates/ziyi.png" alt= "ziyi" style="width:35%; height:65%;">
                    <p> Zi Yi </p>
                    <p> Web Designer </p>
                </div>
                <div class="col-md-6 col-sm-12 col-12 indiv">
                    <img class="teamImg" src="../assets/teammates/hazyra.png" alt= "hazyra" style="width:40%; height:65%;">
                    <p> Hazyra </p>
                    <p> Web Designer </p>
                </div>
            </div>
        </div>
        <br/>
    </div>
</section>

</template>

<script>
import axios from 'axios'

var map;
var service;
// eslint-disable-next-line no-undef
let infowindow = new google.maps.InfoWindow;

export default {

    data() {
        return{
            lat: 0,
            lng: 0, 
            type: "hospital",
            radius: 5000,
            places: [],
            address: "",
            
        }
    },

    beforeCreate() {
        if (window.localStorage.getItem("userInformation") === null) {
            this.$router.push('/login');
        }
    },
    
    mounted() {
        //call DOM, autocomplete object
        // eslint-disable-next-line no-undef
        var options = {
                // eslint-disable-next-line no-undef
                bounds: new google.maps.LatLngBounds(
                    // eslint-disable-next-line no-undef
                    new google.maps.LatLng(1.290270, 103.851959)
                ),
                //search only sg region
                componentRestrictions: {
                    country: "sg"
                },
        }
        // eslint-disable-next-line no-undef
        let autocomplete = new google.maps.places.Autocomplete(
            document.getElementById("autocomplete"), options);

        autocomplete.addListener("place_changed", () =>{
            let place = autocomplete.getPlace();
            this.lat = place.geometry.location.lat()
            this.lng = place.geometry.location.lng()
            this.showUserLocationOnTheMap(this.lat, this.lng);
            // console.log(this.lat);
            // console.log(place);
            // this.showUserLocationOnTheMap(
            //     place.geometry.location.lat(),
            //     place.geometry.location.lng()
            // )
        });
        //initMAP
        // eslint-disable-next-line no-undef
        new google.maps.Map(document.getElementById("map"), {
            // I made it a global so I can use it in out events
            zoom: 10,
            center: { // Set to singapore
                lat: 1.290270,
                lng: 103.851959
            }
        });
    }, //mounted
    methods: {
        userLocationButton() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    position => {
                        console.log(position);
                        this.lat = position.coords.latitude;
                        this.lng = position.coords.longitude;
                        
                        this.getAddressFrom(
                            this.lat,
                            this.lng
                        );
                        //console.log(position.coords.latitude);
                        //console.log(position.coords.longitude);
                        this.showUserLocationOnTheMap(
                            this.lat,
                            this.lng
                        );
                        // this.nearby(this.lat, this.lng)
                    },
                    // eslint-disable-next-line no-unused-vars
                    error => {
                        //this is use when declaring something from the data function
                        this.error = error.message;
                        //console.log(error.message);
                    }
                );
            } else {
                // eslint-disable-next-line no-undef
                this.error = "Your browser does not support geolocation API ";
                //console.log("Your browser does not support geolocation API ");
            }
        },
        getAddressFrom(lat, lng) {
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng="
            + lat +
            ","
            + lng
            + "&key=AIzaSyAQTMFOpjhv6TLGnQRtGA0pCQ9r4W8YKjI")
            .then(response => {
                if(response.data.error_message) {
                    this.error = response.data.error_message;
                    //console.log(response.data.error_message);
                } else {
                    //GEOCODING API
                    this.address = response.data.results[0].formatted_address
                    // console.log(response.data.results[0].formatted_address);
                }
            })
            .catch(error => {
                this.error = error.message;
                // console.log(error.message);
            });
        },
        showUserLocationOnTheMap(latitude, longitude) {
            //Create the Map object
            // eslint-disable-next-line no-undef
            var map = new google.maps.Map(document.getElementById("map"), {
                mapTypeId: "roadmap",
                // eslint-disable-next-line no-undef
                center: new google.maps.LatLng(latitude, longitude),
                zoom: 15
            });
            //Add Marker to user location
            // eslint-disable-next-line no-undef
            new google.maps.Marker({
                // eslint-disable-next-line no-undef
                position: new google.maps.LatLng(latitude, longitude),
                map: map,
                zoom: 6
            })
        },
        initialize() {
            
            // eslint-disable-next-line no-undef
            var pyrmont = new google.maps.LatLng(this.lat,this.lng);

            // eslint-disable-next-line no-undef
            map = new google.maps.Map(document.getElementById('map'), {
            center: pyrmont,
            zoom: 15
            });

            var request = {
                location: pyrmont,
                radius: this.radius,
                type: this.type
            };

            // eslint-disable-next-line no-undef
            service = new google.maps.places.PlacesService(map);
            service.nearbySearch(request, this.callback);
        },
        callback(results, status) {
        // eslint-disable-next-line no-undef
            if (status == google.maps.places.PlacesServiceStatus.OK) {
                for (var i = 0; i < results.length; i++) {
                    this.createMarker(results[i]);
                    // console.log(results[i]);
                }
            }
        },
        createMarker(place) {
            if (!place.geometry || !place.geometry.location) return;

            // eslint-disable-next-line no-undef
            const marker = new google.maps.Marker({
                map,
                position: place.geometry.location,
            });

            // eslint-disable-next-line no-undef
            google.maps.event.addListener(marker, "click", () => {
                // console.log('test');
                // console.log(place);
                infowindow.setContent(`${place.name}, ${place.vicinity}`);
                infowindow.setPosition(place.geometry.location)
                infowindow.open(map);
            });
        }

    }


}//export

</script>

<style>
@font-face {
    font-family: mySecondFont;
    src: url('../fonts/IndieFlower-Regular.ttf');
}

.gradientBackground {
	font-family: mySecondFont;
	background-image: linear-gradient(to bottom right, #d1cef2, #9cadce,#d4afb9);
	position: relative;
}

.main-title {
    text-shadow: 2px 2px black;
    font-size: 400%;
}

.subs {
    font-size: 150%;
    font-weight: bold;
}

.second-main {
    text-shadow: 2px 2px black;
    font-size: 250%;
}

.teamImg{
	align-self: center;
	width: 50%;
	height: 70%;
    /* margin-top: 20px; */
    margin-left: 40px;
    margin-right: 40px;
    margin-bottom:20px;
	z-index:0;
}

.teaminfo {
    font-size: 100%;
    font-weight: bold;
}

.indiv {
    margin-bottom: 40px;
}

</style>