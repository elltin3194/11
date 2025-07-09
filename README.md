# 11import requests
import time
from datetime import datetime
import random

# Weather API configuration (OpenWeatherMap)
API_KEY = "your_api_key_here"  # Replace with your API key
CITY = "New York"  # Change to your city
WEATHER_API = f"http://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}"

# Musical notes and their ASCII representations
NOTES = {
    "C": "o",
    "D": "O",
    "E": "*",
    "F": "+",
    "G": "x",
    "A": "#",
    "B": "@"
}

# Rhythm durations based on time
RHYTHMS = {
    0: "w",  # Whole note (4 beats)
    1: "h",  # Half note (2 beats)
    2: "q",  # Quarter note (1 beat)
    3: "e",  # Eighth note (1/2 beat)
    4: "s",  # Sixteenth note (1/4 beat)
    5: "t"   # Thirty-second note (1/8 beat)
}

def get_weather_data():
    """Fetch current weather data"""
    try:
        response = requests.get(WEATHER_API)
        data = response.json()
        temp = data["main"]["temp"] - 273.15  # Convert to Celsius
        humidity = data["main"]["humidity"]
        weather_desc = data["weather"][0]["description"]
        return temp, humidity, weather_desc
    except:
        # Default values if API fails
        return 20.0, 50, "clear sky"

def generate_melody(seed, length=16):
    """Generate a melody based on a seed value"""
    random.seed(seed)
    notes_list = list(NOTES.keys())
    melody = []
    
    for _ in range(length):
        note = random.choice(notes_list)
        rhythm_idx = random.randint(0, 5)
        rhythm = RHYTHMS[rhythm_idx]
        melody.append((note, rhythm))
    
    return melody

def generate_chords(seed, count=4):
    """Generate chords based on a seed value"""
    random.seed(seed)
    notes_list = list(NOTES.keys())
    chords = []
    
    for _ in range(count):
        root = random.choice(notes_list)
        # Create a triad (root, third, fifth)
        root_idx = notes_list.index(root)
        third_idx = (root_idx + 2) % len(notes_list)
        fifth_idx = (root_idx + 4) % len(notes_list)
        
        chord = [notes_list[root_idx], notes_list[third_idx], notes_list[fifth_idx]]
        chords.append(chord)
    
    return chords

def create_ascii_score(melody, chords, tempo, weather_info):
    """Create an ASCII music score"""
    score = []
    
    # Add title and metadata
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    score.append(f"# Procedural Music Score - Generated at {timestamp}")
    score.append(f"# Weather: {weather_info} | Tempo: {tempo} BPM")
    score.append("")
    
    # Add staff lines
    staff = "=" * 80
    score.append(staff)
    score.append(" " * 30 + "TREBLE CLEF")
    score.append(staff)
    score.append("")
    
    # Add time signature
    score.append(f"Time Signature: 4/4")
    score.append("")
    
    # Add melody
    score.append("Melody:")
    score.append("")
    
    # Create note symbols with rhythm
    note_symbols = []
    for note, rhythm in melody:
        symbol = NOTES[note]
        if rhythm == "w":  # Whole note
            symbol = f"{symbol}---"
        elif rhythm == "h":  # Half note
            symbol = f"{symbol}--"
        elif rhythm == "q":  # Quarter note
            symbol = f"{symbol}-"
        elif rhythm == "e":  # Eighth note
            symbol = f"{symbol}"
        elif rhythm == "s":  # Sixteenth note
            symbol = f"{symbol}/"
        elif rhythm == "t":  # Thirty-second note
            symbol = f"{symbol}//"
        
        note_symbols.append(symbol)
    
    # Split melody into measures of 4 beats
    measures = []
    current_measure = []
    current_beats = 0
    
    for symbol, (_, rhythm) in zip(note_symbols, melody):
        beats = 4 / (2 ** list(RHYTHMS.keys()).index(list(RHYTHMS.values()).index(rhythm)))
        
        if current_beats + beats > 4:
            measures.append(current_measure)
            current_measure = [symbol]
            current_beats = beats
        else:
            current_measure.append(symbol)
            current_beats += beats
    
    if current_measure:
        measures.append(current_measure)
    
    # Format measures
    for i, measure in enumerate(measures):
        measure_str = " | ".join(measure)
        score.append(f"Measure {i+1}: {measure_str}")
    
    score.append("")
    score.append(staff)
    score.append("")
    
    # Add chords
    score.append("Chords:")
    score.append("")
    
    for i, chord in enumerate(chords):
        chord_str = "   ".join([NOTES[note] for note in chord])
        score.append(f"Chord {i+1}: {chord_str}")
    
    score.append("")
    score.append(staff)
    
    return "\n".join(score)

def main():
    # Get current time for seed
    now = datetime.now()
    time_seed = now.hour * 10000 + now.minute * 100 + now.second
    
    # Get weather data
    temp, humidity, weather_desc = get_weather_data()
    weather_seed = int(temp * 10) + humidity
    
    # Combine seeds
    seed = time_seed + weather_seed
    
    # Generate musical elements
    tempo = 60 + int(temp)  # Tempo based on temperature (colder = slower)
    melody = generate_melody(seed)
    chords = generate_chords(seed + 1000)  # Slightly different seed for chords
    
    # Create weather info string
    weather_info = f"{weather_desc}, {temp:.1f}°C, {humidity}% humidity"
    
    # Create ASCII score
    score = create_ascii_score(melody, chords, tempo, weather_info)
    
    # Save to file
    filename = f"music_score_{now.strftime('%Y%m%d_%H%M%S')}.txt"
    with open(filename, 'w') as f:
        f.write(score)
    
    print(f"Generated music score: {filename}")
    print(f"Weather Info: {weather_info}")
    print(f"Tempo: {tempo} BPM")
    print("\nHere's a preview:")
    print("\n".join(score.split("\n")[:20]) + "\n...")

if __name__ == "__main__":
    main()
