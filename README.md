<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="apple-mobile-web-app-capable" content="yes">
        <meta name="apple-mobile-web-app-status-bar-style" content="black">
        <title>MySiteTitle</title>
        <link rel="icon" type="image/x-icon" href="/images/favicon.ico">
        <link rel="apple-touch-icon" href="images/apple-touch-icon-iphone60x60.png">
        <link rel="apple-touch-icon" sizes="60x60" href="images/apple-touchicon-ipad-76x76.png">
        <link rel="apple-touch-icon" sizes="114x114" href="images/apple-touchicon-iphone-retina-120x120.png">
        <link rel="apple-touch-icon" sizes="144x144" href="images/apple-touchicon-ipad-retina-152x152.png">
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    </head>
<body class="vh-100">
    <div class="d-flex h-100 my-auto justify-content-center">
        <div class="card row col-12 col-sm-8 col-md-6 col-xl-4 align-self-center text-center">
            <div class="card-body">
                <div class="row">
                    <div class="col-12 col-sm-6 justify-content-center align-self-center">
                        <i class="bi bi-geo-alt-fill h2 text-danger"></i>
                        <div id="txtCity"></div>
                    </div>
                    <div class="col-12 col-sm-6">
                        <div class="border-bottom">
                            <h5 class="text-secondary bold">Current Conditions</h3>
                            <div class="mb-1">
                                <i id="iconWeather"></i>
                                <span id="txtWeather"></span>
                            </div>
                        </div>
                        <div class="mt-1">
                            <i id="iconTemperature" class="bi bi-thermometer-half"></i>
                            <span id="intTemperature"></span>
                            <i id="iconHumidity" class="bi bi-droplet text-primary"></i>
                            <span id="intHumidity"></span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script>
    <script>
        // gets location coordinates from user. returns array of [latitude,longitude]
        async function getLocationCoords() {
            return new Promise((resolve,reject) => {
                navigator.geolocation.getCurrentPosition((position) => {
                    resolve([position.coords.latitude,position.coords.longitude])
                },(error) => {
                    reject(error.message)
                });
            })
        }

        async function getCityFromCoords(latitude, longitude) {
            const apiKey = 'AIzaSyCz77OFf4qnBGRDsb8EsQA8SAqq7ceEomI'; // Replace with your API key
            const response = await fetch(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${latitude},${longitude}&key=${apiKey}`);
    
            const data = await response.json();
    
            if (data.status === 'OK') {
                const addressComponents = data.results[0].address_components;
                const city = addressComponents.find(component => component.types.includes('locality'));
                return city ? city.long_name : 'City not found';
            } else {
                return 'City not found';
            }
        }

        //main function
        (async() => {
            try {
                var [ latitude, longitude ] = await getLocationCoords()
                var city = await getCityFromCoords(latitude,longitude)
                document.querySelector("#txtCity").textContent = city
                fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,weather_code,relative_humidity_2m`)
                .then(objdata => objdata.json())
                .then((weatherData) => {
                    window.weatherData = weatherData
                    let tempInF = (Math.round(weatherData.current.temperature_2m * (9/5) + 32,0))
                    document.querySelector("#intTemperature").textContent = tempInF + "°F";
                    document.querySelector("#intHumidity").textContent = weatherData.current.relative_humidity_2m

                    // 0	Clear sky
                    // 1	Mainly clear
                    // 2	Partly cloudy
                    // 3	Overcast
                    switch (weatherData.current.weather_code) {
                        case 0:
                            document.querySelector("#iconWeather").className = "bi bi-brightness-high-fill"
                            document.querySelector("#txtWeather").textContent = "Clear sky"
                            break;
                        case 1:
                            document.querySelector("#iconWeather").className = "bi bi-cloud-sun-fill"
                            document.querySelector("#txtWeather").textContent = "Mainly clear"
                            break;
                        case 2:
                            document.querySelector("#iconWeather").className = "bi bi-cloud-fill"
                            document.querySelector("#txtWeather").textContent = "Partly cloudy"
                            break;
                        case 3:
                            document.querySelector("#iconWeather").className = "bi bi-cloud-drizzle-fill"
                            document.querySelector("#txtWeather").textContent = "Overcast"
                            break;
                        default:
                            document.querySelector("#iconWeather").className = "bi bi-patch-question-fill"
                            document.querySelector("#txtWeather").textContent = "Error retreiving"
                            break;
                    }

                    if (tempInF >= 70) {
                        document.querySelector("#iconTemperature").className = "bi bi-thermometer-high text-danger";
                    } else if (tempInF >= 50) {
                        document.querySelector("#iconTemperature").className = "bi bi-thermometer-half";
                    } else if (tempInF >= 33) {
                        document.querySelector("#iconTemperature").className = "bi bi-thermometer-low";
                    } else {
                        document.querySelector("#iconTemperature").className = "bi bi-thermometer-snow text-primary";
                    }
                })
            } catch (error) {
                console.log(error)
            }
        })()
        //THIS APPLICATION IS COURTESY OF OPEN METEO
    </script>
</body>
</html>
