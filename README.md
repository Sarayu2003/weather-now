# weather-now
import React, { useState } from 'react';

// Map city names to background images (you can add more)
const cityBackgrounds = {
  'New York': "url('https://images.unsplash.com/photo-1465101162946-4377e57745c3?auto=format&fit=crop&w=1600&q=80')",
  London: "url('https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1600&q=80')",
  Paris: "url('https://images.unsplash.com/photo-1464983953574-0892a716854b?auto=format&fit=crop&w=1600&q=80')",
  Tokyo: "url('https://images.unsplash.com/photo-1468413253725-0d7857b05f57?auto=format&fit=crop&w=1600&q=80')",
  
};

function WeatherNow() {
  const [city, setCity] = useState('');
  const [weather, setWeather] = useState(null);
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const fetchWeather = async () => {
    if (!city.trim()) {
      setError('Please enter a city name.');
      setWeather(null);
      return;
    }
    setError('');
    setLoading(true);
    try {
      const geoRes = await fetch(
        `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}`
      );
      const geoData = await geoRes.json();
      if (!geoData.results || geoData.results.length === 0) {
        setError('City not found.');
        setWeather(null);
        setLoading(false);
        return;
      }
      const { latitude, longitude, name, country } = geoData.results[0];
      const weatherRes = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`
      );
      const weatherData = await weatherRes.json();
      setWeather({
        ...weatherData.current_weather,
        name,
        country,
      });
    } catch {
      setError('Failed to fetch weather data.');
      setWeather(null);
    } finally {
      setLoading(false);
    }
  };

  // Allow pressing Enter to fetch weather
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') fetchWeather();
  };

  // Determine background image based on city or default
  const bgImage = cityBackgrounds[city.trim()] || cityBackgrounds['New York'];

  return (
    <div
      style={{
        ...styles.wrapper,
        backgroundImage: bgImage,
        backgroundSize: 'cover',
        backgroundPosition: 'center center',
        backgroundRepeat: 'no-repeat',
        transition: 'background-image 0.5s ease-in-out',
      }}
      aria-label="Weather Now Application"
    >
      <div style={styles.overlay}>
        <h1 style={styles.title}>Weather Now</h1>
        <div style={styles.box}>
          <input
            style={styles.input}
            type="text"
            value={city}
            placeholder="Enter city"
            aria-label="City input"
            onChange={(e) => setCity(e.target.value)}
            onKeyDown={handleKeyPress}
          />
          <button
            style={{ ...styles.button, cursor: loading ? 'not-allowed' : 'pointer' }}
            onClick={fetchWeather}
            disabled={loading}
            aria-label="Get weather"
          >
            {loading ? 'Loading...' : 'Get Weather'}
          </button>
        </div>
        {error && (
          <div role="alert" style={styles.error}>
            {error}
          </div>
        )}
        {weather && (
          <div style={styles.card} role="region" aria-live="polite">
            <div style={styles.location}>
              <span style={styles.city}>{weather.name}</span>,{' '}
              <span style={styles.country}>{weather.country}</span>
            </div>
            <div style={styles.stats}>
              <p style={styles.temp}>{weather.temperature}°C</p>
              <p>
                <span style={styles.label}>Windspeed:</span> {weather.windspeed} km/h
              </p>
              <p>
                <span style={styles.label}>Weather code:</span> {weather.weathercode}
              </p>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}

const styles = {
  wrapper: {
    minHeight: '100vh',
    fontFamily: 'Segoe UI, Arial, sans-serif',
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    padding: '1rem',
  },
  overlay: {
    backgroundColor: 'rgba(255, 255, 255, 0.85)',
    borderRadius: '15px',
    padding: '2rem',
    maxWidth: '360px',
    width: '100%',
    boxShadow: '0 8px 24px rgba(0,0,0,0.15)',
    textAlign: 'center',
  },
  title: {
    fontSize: '2.5rem',
    color: '#1577D1',
    marginBottom: '1.5rem',
    fontWeight: '700',
    textShadow: '0 3px 10px rgba(0, 0, 0, 0.07)',
  },
  box: {
    display: 'flex',
    gap: '1rem',
    marginBottom: '1rem',
  },
  input: {
    flex: '1',
    padding: '10px 14px',
    fontSize: '1.125rem',
    borderRadius: '8px',
    border: '2px solid #57A8F9',
    outline: 'none',
    transition: 'border-color 0.3s ease',
  },
  button: {
    padding: '10px 22px',
    borderRadius: '8px',
    backgroundColor: '#1577D1',
    color: '#fff',
    border: 'none',
    fontWeight: '600',
    fontSize: '1.125rem',
    boxShadow: '0 2px 8px rgba(21, 119, 209, 0.3)',
    transition: 'background-color 0.3s ease',
  },
  error: {
    color: '#D11B1B',
    fontWeight: '600',
    marginTop: '0.5rem',
  },
  card: {
    backgroundColor: '#F9FAFB',
    borderRadius: '12px',
    padding: '1.5rem',
    boxShadow: '0 6px 16px rgba(21, 119, 209, 0.1)',
    color: '#1577D1',
  },
  location: {
    fontWeight: '700',
    fontSize: '1.35rem',
    marginBottom: '1rem',
  },
  city: {},
  country: {
    fontWeight: '400',
    color: '#2B2F36',
  },
  stats: {
    fontSize: '1.125rem',
  },
  temp: {
    fontSize: '2.7rem',
    fontWeight: '900',
    marginBottom: '0.75rem',
    color: '#F8AB57',
  },
  label: {
    fontWeight: '600',
    color: '#344054',
  },
};

export default WeatherNow;
