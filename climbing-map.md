---
layout: null
title: Climbing Center Map
---
<div id="map-canvas" style="width:100%; height:650px"></div>

<script src="https://maps.googleapis.com/maps/api/js?v=3.exp"></script>

<script>

    var markers = {{ site.data.climbing | jsonify }};
    
	function initializeMap() {
        var map;
		var bounds = new google.maps.LatLngBounds(),
			mapOptions = {
				mapTypeId: 'roadmap',
				styles: [{"featureType":"water","elementType":"geometry","stylers":[{"visibility":"on"},{"color":"#aee2e0"}]},{"featureType":"landscape","elementType":"geometry.fill","stylers":[{"color":"#abce83"}]},{"featureType":"poi","elementType":"geometry.fill","stylers":[{"color":"#769E72"}]},{"featureType":"poi","elementType":"labels.text.fill","stylers":[{"color":"#7B8758"}]},{"featureType":"poi","elementType":"labels.text.stroke","stylers":[{"color":"#EBF4A4"}]},{"featureType":"poi.park","elementType":"geometry","stylers":[{"visibility":"simplified"},{"color":"#8dab68"}]},{"featureType":"road","elementType":"geometry.fill","stylers":[{"visibility":"simplified"}]},{"featureType":"road","elementType":"labels.text.fill","stylers":[{"color":"#5B5B3F"}]},{"featureType":"road","elementType":"labels.text.stroke","stylers":[{"color":"#ABCE83"}]},{"featureType":"road","elementType":"labels.icon","stylers":[{"visibility":"off"}]},{"featureType":"road.local","elementType":"geometry","stylers":[{"color":"#A4C67D"}]},{"featureType":"road.arterial","elementType":"geometry","stylers":[{"color":"#9BBF72"}]},{"featureType":"road.highway","elementType":"geometry","stylers":[{"color":"#EBF4A4"}]},{"featureType":"transit","stylers":[{"visibility":"off"}]},{"featureType":"administrative","elementType":"geometry.stroke","stylers":[{"visibility":"on"},{"color":"#87ae79"}]},{"featureType":"administrative","elementType":"geometry.fill","stylers":[{"color":"#7f2200"},{"visibility":"off"}]},{"featureType":"administrative","elementType":"labels.text.stroke","stylers":[{"color":"#ffffff"},{"visibility":"on"},{"weight":4.1}]},{"featureType":"administrative","elementType":"labels.text.fill","stylers":[{"color":"#495421"}]},{"featureType":"administrative.neighborhood","elementType":"labels","stylers":[{"visibility":"off"}]}]
            };
            
		// Display a map on the page
		map = new google.maps.Map(document.getElementById("map-canvas"), mapOptions);
        map.setTilt(50);
        
        // Info window content
        var infoWindowContent = [
        {% for item in site.data.climbing %}
        ['<div class="info_content">' + '<h3>{{ item.title }}</h3>' + '<p>{{ item.content }}</p>' + '<p><a href="{{ item.url }}">Website</a></p>' + '</div>'],
        {% endfor %}
        ];

        // Add multiple markers to map
        var infoWindow = new google.maps.InfoWindow(), marker, i;

		for (var i = 0; i < markers.length; i++) {
			var position = new google.maps.LatLng(markers[i].latitude, markers[i].longitude);
			bounds.extend(position);
			marker = new google.maps.Marker({
				position: position,
				map: map,
                title: markers[i].title,
                animation: google.maps.Animation.DROP
            });
        
            // Add info window to marker    
            google.maps.event.addListener(marker, 'click', (function(marker, i) {
            return function() {
                infoWindow.setContent(infoWindowContent[i][0]);
                infoWindow.open(map, marker);
            }
            })(marker, i));

            // Center the map to fit all markers on the screen
            map.fitBounds(bounds);
        }

		// Override our map zoom level once our fitBounds function runs (Make sure it only runs once)
        var boundsListener = google.maps.event.addListener((map), 'bounds_changed', function(event) {
			this.setZoom(10);
			google.maps.event.removeListener(boundsListener);
		});
    
    }
    
    google.maps.event.addDomListener(window, 'load', initializeMap);

</script>