# Weather App Documentation

## Introduction

Welcome to the Weather App! This application allows you to check the current weather in any city. It uses the OpenWeatherMap API to fetch real-time weather data. To get started, you'll need to obtain an API key from OpenWeatherMap.

### Getting OpenWeatherMap API Key

1. Visit [OpenWeatherMap](https://openweathermap.org/) and sign up for a free account.
2. Once logged in, navigate to the "API keys" section in your account dashboard.
3. Generate a new API key and copy it. This key is needed for the Weather App to fetch weather data.

## Weather App Overview

The Weather App is a small web application written in the Go programming language (Golang). It uses the Gin web framework for building the web server and Prometheus for monitoring and metrics.

### Main Features

1. **Weather Information:** Retrieve real-time weather data for a specified city.
2. **Prometheus Metrics:** Track the number of requests and page views to monitor application usage.

## Code Overview

The Weather App consists of three main files: `main.go`, `metrics.go`, and HTML templates (`index.html` and `weather.html`). Additionally, there's a stylesheet (`styles.css`) for basic styling.

### `main.go`

#### WeatherData Struct
```go
type WeatherData struct {
	Main struct {
		Temp     float64 `json:"temp"`
		Humidity int     `json:"humidity"`
	} `json:"main"`
	Weather []struct {
		Description string `json:"description"`
		Icon        string `json:"icon"`
	} `json:"weather"`
}
```
Example JSON response structure from the OpenWeatherMap API:
```json
{
  "main": {
    "temp": 23.5,
    "humidity": 60
  },
  "weather": [
    {
      "description": "Clear sky",
      "icon": "01d"
    }
  ]
}
```

#### Middleware
```go
router.Use(MetricsMiddleware())
```
This middleware increments counters for each request, tracking the total number of requests and page views.

#### Endpoints
```go
router.GET("/", func(c *gin.Context) {
  // ... (HTML response for the main page)
})

router.GET("/weather", func(c *gin.Context) {
  // ... (Fetches weather data from OpenWeatherMap API and responds with HTML)
})

router.GET("/metrics", MetricsHandler)
```
These define the main routes:
- `/`: Displays the main page.
- `/weather`: Fetches weather data and displays it.
- `/metrics`: Exposes Prometheus metrics.

### `metrics.go`

#### Prometheus Counters
```go
var (
  totalRequests = prometheus.NewCounterVec(
    prometheus.CounterOpts{
      Name: "go_total_requests",
      Help: "Total number of requests to the web server",
    },
    []string{},
  )

  pageViews = prometheus.NewCounterVec(
    prometheus.CounterOpts{
      Name: "web_page_views_total",
      Help: "Total number of page views",
    },
    []string{"path"},
  )
)
```
Example Prometheus metrics generated:
```
go_total_requests{job="weather_app"} 42
web_page_views_total{job="weather_app", path="/weather"} 20
```

#### MetricsMiddleware
```go
func MetricsMiddleware() gin.HandlerFunc {
  // ... (Increments counters for each request)
}
```

#### MetricsHandler
```go
func MetricsHandler(c *gin.Context) {
  promhttp.Handler().ServeHTTP(c.Writer, c.Request)
}
```
Handles the `/metrics` endpoint, exposing the Prometheus metrics.

### HTML Templates

#### `index.html`
```html
<form id="cityForm">
  <!-- ... (Input form for the city) -->
</form>
```

#### `weather.html`
```html
<p>Temperature: <span id="temperature">{{printf "%.2f" .temperature}}</span> &#8451;</p>
```
Displays the temperature with two decimal places.

### `styles.css`

#### Styling
```css
.container {
  /* ... (Defines styling for the container) */
}

button {
  /* ... (Styling for the submit button) */
}
```

These are just highlights, and you can explore the entire codebase for more details.

## Running the Application

1. Ensure you have Golang installed.
2. Clone the repository.
3. Create a `.env` file in the project root with your OpenWeatherMap API key:
```bash
OPENWEATHERMAP_API_KEY=your_api_key_here
```
Run the application:
```bash
go run .
```
Access the Weather App in your browser at `http://localhost:8080`.

## Prometheus Metrics

The Weather App leverages Prometheus, an open-source monitoring and alerting toolkit, to track and expose relevant metrics about its performance. Two key metrics have been implemented to provide insights into the application's behavior:

### 1. `go_total_requests`

- **Description:** Counts the total number of requests made to the web server.
- **Usage:** Monitors the overall traffic and usage of the Weather App.
- **Prometheus Metric Query Example:**
  ```promql
  go_total_requests{job="weather_app"}
  ```

### 2. `web_page_views_total`

- **Description:** Tracks the total number of page views for each specific path.
- **Usage:** Helps understand which pages are most frequently accessed.
- **Prometheus Metric Query Example:**
  ```promql
  web_page_views_total{job="weather_app", path="/weather"}
  ```

These metrics provide valuable insights into the application's performance and user interaction. By querying these metrics, you can gain a better understanding of how users are engaging with the Weather App and identify potential areas for improvement or optimization. The integration of Prometheus metrics enhances the application's observability and contributes to a more robust monitoring infrastructure.

## Conclusion

The Weather App is a simple yet effective tool for checking the weather in any city. With its clean design and integration of the OpenWeatherMap API, it provides real-time weather information. Additionally, the use of Prometheus metrics allows for monitoring and tracking the application's usage, ensuring a smooth and reliable experience for users. Whether you're interested in the codebase or exploring weather data, the Weather App offers an accessible and informative solution.
