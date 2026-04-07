# gomove-app2
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import MapView, { Marker, Polyline } from 'react-native-maps';
import * as Location from 'expo-location';

export default function App() {
  const [location, setLocation] = useState(null);
  const [drivers, setDrivers] = useState([]);
  const [selectedDriver, setSelectedDriver] = useState(null);
  const [routeCoords, setRouteCoords] = useState([]);
  const [status, setStatus] = useState('idle');

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') return;

      let loc = await Location.getCurrentPositionAsync({});
      setLocation(loc.coords);

      const mockDrivers = [
        { id: 1, latitude: loc.coords.latitude + 0.01, longitude: loc.coords.longitude + 0.01 },
        { id: 2, latitude: loc.coords.latitude - 0.008, longitude: loc.coords.longitude + 0.005 },
        { id: 3, latitude: loc.coords.latitude + 0.005, longitude: loc.coords.longitude - 0.009 }
      ];

      setDrivers(mockDrivers);
    })();
  }, []);

  const findNearestDriver = () => {
    if (!drivers.length || !location) return null;

    let nearest = drivers[0];
    let minDist = Infinity;

    drivers.forEach(driver => {
      const dist = Math.hypot(
        driver.latitude - location.latitude,
        driver.longitude - location.longitude
      );

      if (dist < minDist) {
        minDist = dist;
        nearest = driver;
      }
    });

    return nearest;
  };

  const updateRoute = (driver) => {
    if (!driver || !location) return;

    setRouteCoords([
      { latitude: driver.latitude, longitude: driver.longitude },
      { latitude: location.latitude, longitude: location.longitude }
    ]);
  };

  const handleRequestRide = () => {
    const driver = findNearestDriver();
    setSelectedDriver(driver);
    updateRoute(driver);
    setStatus('driver_on_the_way');
  };

  if (!location) {
    return <Text>Carregando localização...</Text>;
  }

  return (
    <View style={{ flex: 1 }}>
      <MapView
        style={{ flex: 1 }}
        region={{
          latitude: location.latitude,
          longitude: location.longitude,
          latitudeDelta: 0.01,
          longitudeDelta: 0.01,
        }}
      >
        <Marker coordinate={location} title="Você" />

        {drivers.map(driver => (
          <Marker
            key={driver.id}
            coordinate={{ latitude: driver.latitude, longitude: driver.longitude }}
            title={`Motorista ${driver.id}`}
            pinColor={selectedDriver?.id === driver.id ? 'blue' : 'green'}
          />
        ))}

        {routeCoords.length > 0 && (
          <Polyline coordinates={routeCoords} strokeWidth={4} strokeColor="blue" />
        )}
      </MapView>

      <View style={{
        position: 'absolute',
        bottom: 40,
        left: 20,
        right: 20,
        backgroundColor: 'white',
        padding: 20,
        borderRadius: 12
      }}>
        <Text>GoMove Status: {status}</Text>

        <TouchableOpacity
          onPress={handleRequestRide}
          style={{ backgroundColor: 'black', padding: 15, marginTop: 10 }}
        >
          <Text style={{ color: 'white', textAlign: 'center' }}>
            Chamar GoMove
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
