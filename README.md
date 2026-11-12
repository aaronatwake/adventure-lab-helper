import React, { useEffect, useMemo, useState } from 'react';
import {
  Alert,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
  StyleSheet,
  Text,
  TextInput,
  TouchableOpacity,
  View,
} from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as Clipboard from 'expo-clipboard';
import * as Location from 'expo-location';

const STORAGE_KEY = '@adventurelab:stops';

type LabStop = {
  id: string;
  title: string;
  clue: string;
  answer: string;
  notes: string;
  latitude: number | null;
  longitude: number | null;
};

const initialStops: LabStop[] = [
  {
    id: '1',
    title: 'Stop 1',
    clue: 'Find the landmark with the oldest clock in town.',
    answer: '',
    notes: '',
    latitude: null,
    longitude: null,
  },
  {
    id: '2',
    title: 'Stop 2',
    clue: 'Look for the bridge with the curved arches.',
    answer: '',
    notes: '',
    latitude: null,
    longitude: null,
  },
];

const blankStop = (): LabStop => ({
  id: Date.now().toString(),
  title: `Stop ${Date.now()}`,
  clue: '',
  answer: '',
  notes: '',
  latitude: null,
  longitude: null,
});

export default function App() {
  const [stops, setStops] = useState<LabStop[]>(initialStops);
  const [isLoaded, setIsLoaded] = useState(false);

  useEffect(() => {
    const load = async () => {
      try {
        const raw = await AsyncStorage.getItem(STORAGE_KEY);
        if (raw) {
          const parsed = JSON.parse(raw) as LabStop[];
          setStops(parsed.length ? parsed : initialStops);
        }
      } catch (error) {
        console.warn('Could not load saved stops:', error);
      } finally {
        setIsLoaded(true);
      }
    };

    load();
  }, []);

  useEffect(() => {
    if (!isLoaded) {
      return;
    }

    const save = async () => {
      try {
        await AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(stops));
      } catch (error) {
        console.warn('Could not save stops:', error);
      }
    };

    save();
  }, [stops, isLoaded]);

  const answeredCount = useMemo(
    () => stops.filter((stop) => stop.answer.trim().length > 0).length,
    [stops],
  );

  const updateStop = (id: string, field: keyof LabStop, value: string | number | null) => {
    setStops((current) =>
      current.map((stop) => (stop.id === id ? { ...stop, [field]: value } : stop)),
    );
  };

  const addStop = () => {
    setStops((current) => [blankStop(), ...current]);
  };

  const removeStop = (id: string) => {
    setStops((current) => current.filter((stop) => stop.id !== id));
  };

  const useMyLocation = async (id: string) => {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Location permission denied', 'The app needs location access to tag stops.');
        return;
      }

      const location = await Location.getCurrentPositionAsync({ accuracy: Location.Accuracy.Low });
      updateStop(id, 'latitude', location.coords.latitude);
      updateStop(id, 'longitude', location.coords.longitude);
      Alert.alert('Location saved', 'Your current coordinates have been added to this stop.');
    } catch (error) {
      console.warn('Location update failed:', error);
      Alert.alert('Location failed', 'We could not get your current location right now.');
    }
  };

  const copyAnswer = async (stop: LabStop) => {
    if (!stop.answer.trim()) {
      Alert.alert('No answer entered', 'Add the answer before copying.');
      return;
    }

    await Clipboard.setStringAsync(stop.answer.trim());
    Alert.alert('Answer copied', `${stop.title} was copied to the clipboard.`);
  };

  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
      style={styles.container}
    >
      <ScrollView contentContainerStyle={styles.scrollContent}>
        <View style={styles.headerRow}>
          <View>
            <Text style={styles.eyebrow}>ADVENTURE LAB</Text>
            <Text style={styles.title}>Helper</Text>
          </View>
          <View style={styles.progressPill}>
            <Text style={styles.progressText}>{answeredCount}/{stops.length}</Text>
          </View>
        </View>

        <Text style={styles.subtitle}>
          Save clues, answers, notes, and locations while you work through the trail.
        </Text>

        <TouchableOpacity style={styles.primaryButton} onPress={addStop}>
          <Text style={styles.primaryButtonText}>+ Add stop</Text>
        </TouchableOpacity>

        {stops.map((stop, index) => (
          <View key={stop.id} style={styles.card}>
            <View style={styles.cardHeader}>
              <Text style={styles.cardTitle}>{stop.title || `Stop ${index + 1}`}</Text>
              <TouchableOpacity onPress={() => removeStop(stop.id)}>
                <Text style={styles.removeText}>Remove</Text>
              </TouchableOpacity>
            </View>

            <Text style={styles.label}>Label</Text>
            <TextInput
              style={styles.input}
              value={stop.title}
              onChangeText={(text) => updateStop(stop.id, 'title', text)}
              placeholder="Stop name"
            />

            <Text style={styles.label}>Clue</Text>
            <TextInput
              style={[styles.input, styles.textArea]}
              multiline
              value={stop.clue}
              onChangeText={(text) => updateStop(stop.id, 'clue', text)}
              placeholder="Write down the clue text"
            />

            <Text style={styles.label}>Answer</Text>
            <TextInput
              style={[styles.input, styles.textArea]}
              multiline
              value={stop.answer}
              onChangeText={(text) => updateStop(stop.id, 'answer', text)}
              placeholder="Type the answer you think fits"
            />

            <Text style={styles.label}>Notes</Text>
            <TextInput
              style={[styles.input, styles.textArea]}
              multiline
              value={stop.notes}
              onChangeText={(text) => updateStop(stop.id, 'notes', text)}
              placeholder="Add your notes, hints, or observations"
            />

            <View style={styles.locationRow}>
              <Text style={styles.locationText}>
                {stop.latitude !== null && stop.longitude !== null
                  ? `Saved: ${stop.latitude.toFixed(4)}, ${stop.longitude.toFixed(4)}`
                  : 'No location saved yet'}
              </Text>
              <TouchableOpacity style={styles.secondaryButton} onPress={() => useMyLocation(stop.id)}>
                <Text style={styles.secondaryButtonText}>Use my location</Text>
              </TouchableOpacity>
            </View>

            <TouchableOpacity style={styles.copyButton} onPress={() => copyAnswer(stop)}>
              <Text style={styles.copyButtonText}>Copy answer</Text>
            </TouchableOpacity>
          </View>
        ))}
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#09111f',
  },
  scrollContent: {
    padding: 20,
    paddingBottom: 48,
  },
  headerRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8,
  },
  eyebrow: {
    color: '#8cc5ff',
    fontSize: 12,
    letterSpacing: 1.4,
    fontWeight: '700',
  },
  title: {
    color: '#f4f7fa',
    fontSize: 34,
    fontWeight: '800',
  },
  progressPill: {
    backgroundColor: '#1b2b43',
    borderRadius: 999,
    paddingHorizontal: 12,
    paddingVertical: 8,
  },
  progressText: {
    color: '#ecf4ff',
    fontWeight: '700',
  },
  subtitle: {
    color: '#b7c9da',
    fontSize: 14,
    lineHeight: 20,
    marginBottom: 20,
  },
  primaryButton: {
    backgroundColor: '#3fa0ff',
    borderRadius: 12,
    paddingVertical: 14,
    alignItems: 'center',
    marginBottom: 18,
  },
  primaryButtonText: {
    color: '#fff',
    fontWeight: '700',
    fontSize: 16,
  },
  card: {
    backgroundColor: '#12243d',
    borderRadius: 18,
    padding: 16,
    marginBottom: 16,
    borderWidth: 1,
    borderColor: '#21456e',
  },
  cardHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 12,
  },
  cardTitle: {
    color: '#f4f7fa',
    fontSize: 20,
    fontWeight: '700',
  },
  removeText: {
    color: '#ff8c8c',
    fontWeight: '600',
  },
  label: {
    color: '#d8e7f7',
    fontSize: 12,
    fontWeight: '700',
    marginBottom: 8,
    marginTop: 10,
  },
  input: {
    backgroundColor: '#0d1b2c',
    paddingHorizontal: 12,
    paddingVertical: 12,
    borderRadius: 10,
    borderWidth: 1,
    borderColor: '#20375a',
    color: '#ecf4ff',
  },
  textArea: {
    minHeight: 80,
    textAlignVertical: 'top',
  },
  locationRow: {
    marginTop: 12,
  },
  locationText: {
    color: '#bfd6f2',
    fontSize: 12,
    marginBottom: 10,
  },
  secondaryButton: {
    backgroundColor: '#1a324d',
    borderRadius: 10,
    paddingVertical: 10,
    alignItems: 'center',
  },
  secondaryButtonText: {
    color: '#dceeff',
    fontWeight: '700',
  },
  copyButton: {
    backgroundColor: '#2d7f5d',
    borderRadius: 10,
    paddingVertical: 12,
    alignItems: 'center',
    marginTop: 12,
  },
  copyButtonText: {
    color: '#ebfff5',
    fontWeight: '700',
  },
});
