// Replace YOUR_API_KEY with your actual Google Maps API key
const apiKey = "YOUR_API_KEY";

// Initialize Google Map
function initMap() {
    const map = new google.maps.Map(document.getElementById("map"), {
        zoom: 15,
        center: { lat: 22.3027, lng: 114.1772 }, // Default center to Hong Kong
    });
}

// Search for route
$("#search-route").click(function () {
    const destination = $("#destination").val();

    // Use Google Maps API to search for route
    const directionsService = new google.maps.DirectionsService();
    const directionsRenderer = new google.maps.DirectionsRenderer();
    directionsRenderer.setMap(map);

    const request = {
        origin: { lat: 22.3027, lng: 114.1772 }, // Replace with user's current location
        destination: destination,
        travelMode: google.maps.TravelMode.TRANSIT,
    };

    directionsService.route(request, function (response, status) {
        if (status === "OK") {
            // Display route details
            const route = response.routes[0];
            const legs = route.legs;
            let instructions = "";

            for (let i = 0; i < legs.length; i++) {
                const steps = legs[i].steps;
                for (let j = 0; j < steps.length; j++) {
                    instructions += `${steps[j].instructions} `;
                }
            }

            $("#route-details").html(instructions);

            // Draw route on map
            directionsRenderer.setDirections(response);
        } else {
            alert("無法找到路線");
        }
    });
});

// Start navigation
let userMarker;
let watchId;

$("#start-navigation").click(function () {
    // Start tracking user's location
    if (navigator.geolocation) {
        watchId = navigator.geolocation.watchPosition(function (position) {
            const lat = position.coords.latitude;
            const lng = position.coords.longitude;

            // Update user's location on map
            if (!userMarker) {
                userMarker = new google.maps.Marker({
                    position: { lat, lng },
                    map: map,
                    icon: "https://maps.google.com/mapfiles/ms/icons/blue-dot.png",
                });
            } else {
                userMarker.setPosition({ lat, lng });
            }

            // Check for path deviation
            // ...
        });
    } else {
        alert("您的瀏覽器不支援定位功能");
    }
});

// Voice input (using Google Cloud Speech-to-Text API)
$("#voice-input").click(function () {
    if (!('webkitSpeechRecognition' in window)) {
        alert("您的瀏覽器不支援語音輸入");
        return;
    }

    const recognition = new webkitSpeechRecognition();
    recognition.lang = "yue-Hant-HK"; // Cantonese (Traditional Chinese, Hong Kong)
    recognition.continuous = true;
    recognition.interimResults = true;

    recognition.onresult = function (event) {
        const transcript = event.results[event.resultIndex][0].transcript;
        $("#destination").val(transcript);
    };

    recognition.start();
});

// Path deviation alert
let lastKnownPosition;

function checkPathDeviation(position) {
    const lat = position.coords.latitude;
    const lng = position.coords.longitude;

    // Calculate distance from last known position
    if (lastKnownPosition) {
        const distance = google.maps.geometry.spherical.computeDistanceBetween(
            new google.maps.LatLng(lat, lng),
            new google.maps.LatLng(lastKnownPosition.latitude, lastKnownPosition.longitude)
        );

        if (distance > 300) {
            alert("您可能已偏離路線，可能需要重新規劃路線");
        }
    }

    lastKnownPosition = { latitude: lat, longitude: lng };
}

// Cantonese text-to-speech (using Google Cloud Text-to-Speech API)
function speakCantonese(text) {
    // ... (Implementation using Google Cloud Text-to-Speech API)
}

// Event listener for clicking on instructions
$("#route-details").on("click", "p", function () {
    const instruction = $(this).text();
    speakCantonese(instruction);
});
// Import the Google Cloud client library
const textToSpeech = require('@google-cloud/text-to-speech');

// Create a Text-to-Speech client
const client = new textToSpeech.TextToSpeechClient();

async function speakCantonese(text) {
  // Construct the request
  const request = {
    input: { text: text },
    // Voice configuration
    voice: { languageCode: 'yue-HK', ssmlGender: 'NEUTRAL' },
    audioConfig: { audioEncoding: 'MP3' },
  };

  // Perform the Text-to-Speech request
  const [response] = await client.synthesizeSpeech(request);
  // Write the binary audio content to a local file
  const writeFile = require('fs').writeFile;
  writeFile('output.mp3', response.audioContent, 'binary', err => {
    if (err) {
      console.error('Error writing audio file:', err);
    }
  });

  // Play the audio file
  const playSound = require('play-sound')();
  playSound('output.mp3');
}
