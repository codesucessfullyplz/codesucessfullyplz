<!DOCTYPE html>
<html>
<head>
    <title>Find Restaurants Nearby</title>
    <style>
        #map {
            height: 400px;
            width: 100%;
        }
    </style>
</head>
<body>
    <div>
        <label for="type">Type:</label>
        <select id="type">
            <option value="">Any</option>
            <option value="chinese">Chinese</option>
            <option value="indian">Indian</option>
            <!-- Add more options as needed -->
        </select>

        <label for="price">Price Level:</label>
        <select id="price">
            <option value="">Any</option>
            <option value="0">Free</option>
            <option value="1">$</option>
            <option value="2">$$</option>
            <option value="3">$$$</option>
            <option value="4">$$$$</option>
        </select>

        <button onclick="findRestaurants()">Find Restaurants</button>
    </div>

    <div id="map"></div>

    <script>
        let map;
        let service;
        let infowindow;
        let currentLocation;
        let markers = [];

        function initMap() {
            infowindow = new google.maps.InfoWindow();

            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition((position) => {
                    currentLocation = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude
                    };

                    map = new google.maps.Map(document.getElementById('map'), {
                        center: currentLocation,
                        zoom: 14
                    });

                    service = new google.maps.places.PlacesService(map);

                    // Automatically find nearby restaurants on load
                    findRestaurants();
                }, () => {
                    handleLocationError(true, infowindow, map.getCenter());
                });
            } else {
                // Browser doesn't support Geolocation
                handleLocationError(false, infowindow, map.getCenter());
            }
        }

        function findRestaurants() {
            if (!currentLocation) {
                console.error("Current location not set.");
                return;
            }

            const type = document.getElementById('type').value;
            const price = document.getElementById('price').value;

            const request = {
                location: currentLocation,
                radius: '5000',
                type: ['restaurant'],
                keyword: type,
                minPriceLevel: price ? parseInt(price) : undefined,
                maxPriceLevel: price ? parseInt(price) : undefined,
            };

            service.nearbySearch(request, (results, status, pagination) => {
                if (status === google.maps.places.PlacesServiceStatus.OK) {
                    clearMarkers();
                    for (let i = 0; i < results.length; i++) {
                        createMarker(results[i]);
                    }
                    if (pagination.hasNextPage) {
                        const moreButton = document.getElementById('more');
                        moreButton.disabled = false;
                        moreButton.addEventListener('click', () => {
                            moreButton.disabled = true;
                            pagination.nextPage();
                        });
                    }
                } else {
                    console.error('Error in nearbySearch:', status);
                }
            });
        }

        function createMarker(place) {
            const placeLoc = place.geometry.location;
            const marker = new google.maps.Marker({
                map: map,
                position: placeLoc
            });
            markers.push(marker);

            google.maps.event.addListener(marker, 'click', () => {
                infowindow.setContent(place.name + '<br>' + place.vicinity);
                infowindow.open(map, marker);
            });
        }

        function clearMarkers() {
            for (let i = 0; i < markers.length; i++) {
                markers[i].setMap(null);
            }
            markers = [];
        }

        function handleLocationError(browserHasGeolocation, infoWindow, pos) {
            infoWindow.setPosition(pos);
            infoWindow.setContent(browserHasGeolocation ?
                'Error: The Geolocation service failed.' :
                'Error: Your browser doesn\'t support geolocation.');
            infoWindow.open(map);
        }

        window.onload = initMap;
    </script>
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places&callback=initMap" async defer></script>
</body>
</html>


