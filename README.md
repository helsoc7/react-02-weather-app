## React Wetter App
1. API-Registrierung
Wir müssen uns erstmal bei einer Wetter-API registrieren. Ich habe mich für https://weatherstack.com/ entschieden. Dort loggst du dich einfach ein, und generierst einen API-Schlüssel
2. Projekt erstellen
```
npx create-react-app weather-app
cd weather-app
```
3. App.js Komponente erstellen
```
import React, { useState, useEffect } from 'react';


const API_KEY = process.env.REACT_APP_API_KEY;

function App() {
  const [weather, setWeather] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  const [city, setCity] = useState('');

  const fetchWeatherData = async (lat, lon) => {
    try {
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`
      );
      if (!response.ok) throw new Error('Fehler beim Abrufen der Wetterdaten');
      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    const loadWeatherByLocation = () => {
      setLoading(true);
      navigator.geolocation.getCurrentPosition(
        async (position) => {
          const { latitude, longitude } = position.coords;
          await fetchWeatherData(latitude, longitude);
        },
        (err) => {
          console.error(err); 
          setError(`Fehlercode: ${err.code}, Nachricht: ${err.message}`);
          setLoading(false);
        }
      );
    };

    loadWeatherByLocation();
  }, []);

  const handleCityInputChange = (e) => setCity(e.target.value);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError(null);
    try {
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
      );
      if (!response.ok) throw new Error('Stadt nicht gefunden');
      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container">
      <h1>Weather App</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Enter city name"
          value={city}
          onChange={handleCityInputChange}
        />
        <button type="submit">Search</button>
      </form>
      {loading && <p>Loading...</p>}
      {error && <p>{error}</p>}
      {weather && (
        <div>
          <h2>{weather.name}</h2>
          <p>Temperature: {weather.main.temp}°C</p>
          <p>Description: {weather.weather[0].description}</p>
          <p>Wind Speed: {weather.wind.speed} m/s</p>
        </div>
      )}
    </div>
  );
}

export default App;
```
4. .env Datei erstellen
```
REACT_APP_API_KEY='Dein KEy'
```
Achtung: Wenn wir Umgebungsvariablen in React einbinden, dann muss die Variable immer mit REACT_APP beginnen. Das dotenv-Paket wird automatisch beim Initialisieren des Projekts installiert.
