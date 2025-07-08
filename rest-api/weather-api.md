# Weather API Documentation

## Overview
The Weather API provides real-time weather data and forecasts for locations worldwide. Access current conditions, hourly forecasts, and 7-day predictions with a simple REST API.

## Base URL
```
https://api.weatherservice.com/v2
```

## Authentication

All requests require an API key passed in the header:

```bash
curl -H "X-API-Key: YOUR_API_KEY" https://api.weatherservice.com/v2/current
```

To get an API key, sign up at [developer.weatherservice.com](https://developer.weatherservice.com)

## Rate Limiting

| Plan | Requests/Day | Burst Limit |
|------|--------------|-------------|
| Free | 1,000 | 10/minute |
| Pro | 10,000 | 100/minute |
| Enterprise | Unlimited | 1000/minute |

Rate limit information is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1719432000
```

## Endpoints

### Current Weather

Get current weather conditions for a location.

**GET** `/current`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| location | string | Yes* | City name, coordinates, or zip code |
| lat | float | Yes* | Latitude (-90 to 90) |
| lon | float | Yes* | Longitude (-180 to 180) |
| units | string | No | Temperature units: `metric`, `imperial` (default: `metric`) |

*Either `location` OR `lat`+`lon` required

#### Example Request
```bash
# By city name
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/current?location=London"

# By coordinates  
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/current?lat=51.5074&lon=-0.1278"
```

#### Example Response
```json
{
  "location": {
    "name": "London",
    "country": "UK",
    "lat": 51.5074,
    "lon": -0.1278,
    "timezone": "Europe/London"
  },
  "current": {
    "temp": 18.5,
    "feels_like": 17.2,
    "humidity": 72,
    "pressure": 1013,
    "visibility": 10000,
    "wind": {
      "speed": 5.2,
      "direction": 230,
      "gust": 8.1
    },
    "weather": {
      "main": "Clouds",
      "description": "Partly cloudy",
      "icon": "02d"
    },
    "dt": 1719420845
  }
}
```

### Weather Forecast

Get weather forecast for up to 7 days.

**GET** `/forecast`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| location | string | Yes* | City name, coordinates, or zip code |
| lat | float | Yes* | Latitude |
| lon | float | Yes* | Longitude |
| days | integer | No | Number of days (1-7, default: 7) |
| units | string | No | Temperature units |

#### Example Request
```bash
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/forecast?location=London&days=3"
```

#### Example Response
```json
{
  "location": {
    "name": "London",
    "country": "UK",
    "lat": 51.5074,
    "lon": -0.1278
  },
  "forecast": [
    {
      "date": "2024-06-26",
      "temp": {
        "min": 14.2,
        "max": 22.8,
        "avg": 18.5
      },
      "humidity": 65,
      "wind": {
        "speed": 4.8,
        "direction": 220
      },
      "weather": {
        "main": "Clear",
        "description": "Sunny",
        "icon": "01d"
      },
      "precipitation": {
        "probability": 10,
        "amount": 0
      }
    },
    {
      "date": "2024-06-27",
      "temp": {
        "min": 15.1,
        "max": 21.3,
        "avg": 18.2
      },
      "humidity": 70,
      "wind": {
        "speed": 5.5,
        "direction": 240
      },
      "weather": {
        "main": "Rain",
        "description": "Light rain",
        "icon": "10d"
      },
      "precipitation": {
        "probability": 80,
        "amount": 2.5
      }
    }
  ]
}
```

### Search Locations

Search for locations to get accurate coordinates.

**GET** `/search`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Search query |
| limit | integer | No | Max results (1-10, default: 5) |

#### Example Request
```bash
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/search?query=London"
```

#### Example Response
```json
{
  "results": [
    {
      "name": "London",
      "country": "United Kingdom",
      "state": "England",
      "lat": 51.5074,
      "lon": -0.1278
    },
    {
      "name": "London",
      "country": "Canada",
      "state": "Ontario", 
      "lat": 42.9849,
      "lon": -81.2497
    }
  ]
}
```

## Error Handling

The API returns standard HTTP status codes and JSON error responses.

### Error Response Format
```json
{
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid",
    "details": {
      "provided_key": "xxx...xxx"
    }
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| INVALID_API_KEY | 401 | API key is missing or invalid |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| INVALID_LOCATION | 400 | Location not found or invalid |
| MISSING_PARAMETER | 400 | Required parameter missing |
| SERVER_ERROR | 500 | Internal server error |

### Error Examples

```bash
# Missing API key
{
  "error": {
    "code": "INVALID_API_KEY",
    "message": "API key is required"
  }
}

# Invalid location
{
  "error": {
    "code": "INVALID_LOCATION", 
    "message": "Could not find location: 'InvalidCity123'"
  }
}
```

## Code Examples

### JavaScript (Node.js)
```javascript
const axios = require('axios');

const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.weatherservice.com/v2';

async function getCurrentWeather(location) {
  try {
    const response = await axios.get(`${BASE_URL}/current`, {
      headers: { 'X-API-Key': API_KEY },
      params: { location }
    });
    
    const { temp, weather } = response.data.current;
    console.log(`${location}: ${temp}°C, ${weather.description}`);
    
  } catch (error) {
    console.error('Error:', error.response.data.error.message);
  }
}

getCurrentWeather('London');
```

### Python
```python
import requests

API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.weatherservice.com/v2'

def get_weather_forecast(location, days=3):
    headers = {'X-API-Key': API_KEY}
    params = {'location': location, 'days': days}
    
    response = requests.get(f'{BASE_URL}/forecast', 
                          headers=headers, 
                          params=params)
    
    if response.status_code == 200:
        data = response.json()
        for day in data['forecast']:
            print(f"{day['date']}: {day['weather']['description']}")
    else:
        error = response.json()['error']
        print(f"Error: {error['message']}")

get_weather_forecast('London', days=3)
```

### cURL
```bash
# Get current weather
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/current?location=London"

# Get 5-day forecast
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/forecast?location=London&days=5"

# Search for locations
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.weatherservice.com/v2/search?query=London&limit=3"
```

## Webhooks

Subscribe to weather alerts and updates via webhooks.

### Webhook Setup
```json
POST /webhooks
{
  "url": "https://your-server.com/weather-webhook",
  "events": ["severe_weather", "daily_forecast"],
  "location": "London"
}
```

### Webhook Payload
```json
{
  "event": "severe_weather",
  "location": "London",
  "alert": {
    "type": "storm_warning",
    "severity": "moderate",
    "message": "Thunderstorms expected",
    "valid_until": "2024-06-26T18:00:00Z"
  },
  "timestamp": "2024-06-26T12:00:00Z"
}
```

## SDKs

Official SDKs available:
- JavaScript/Node.js: `npm install @weatherservice/sdk`
- Python: `pip install weatherservice`
- PHP: `composer require weatherservice/sdk`
- Ruby: `gem install weatherservice`

## Best Practices

1. **Cache responses** - Weather data doesn't change every second. Cache for 10-15 minutes.
2. **Use location IDs** - After searching, store location coordinates for consistent results.
3. **Handle errors gracefully** - Always check for error responses before processing data.
4. **Batch requests** - Use the forecast endpoint instead of multiple current weather calls.
5. **Monitor rate limits** - Check headers and implement exponential backoff.

## Support

- Documentation: [docs.weatherservice.com](https://docs.weatherservice.com)
- Status: [status.weatherservice.com](https://status.weatherservice.com)
- Support: support@weatherservice.com
- Community: [community.weatherservice.com](https://community.weatherservice.com)

---

*Last updated: June 2024 | API Version: 2.0*
