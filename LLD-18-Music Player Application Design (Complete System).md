# LLD-16: Music Player Application Design (Complete System)

---

## 📋 Table of Contents
- [Introduction](#introduction)
- [Requirements Analysis](#requirements-analysis)
- [System Overview](#system-overview)
- [Design Patterns Used](#design-patterns-used)
- [Bottom-Up Design Approach](#bottom-up-design-approach)
- [Core Components](#core-components)
- [Device Management System](#device-management-system)
- [Playlist Management System](#playlist-management-system)
- [Playing Strategy System](#playing-strategy-system)
- [Facade Layer](#facade-layer)
- [Complete UML Diagram](#complete-uml-diagram)
- [Implementation](#implementation)
- [Key Design Decisions](#key-design-decisions)
- [Real-World Applications](#real-world-applications)
- [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Aaj hum ek **Music Player Application** design karenge - ek complete real-world LLD problem jo interviews mein bahut common hai. Ye problem bahut interesting hai kyunki:

1. **Multiple Design Patterns** ek saath use hote hain
2. **Clean Code Principles** ka practical application dekhte hain
3. **SOLID Principles** ko follow karte hain
4. **Scalable and Extensible** design banate hain

**Key Learning:**
- Kaise different design patterns interact karte hain
- Kaise managers ko use karke code organize karte hain
- Kaise facade pattern se complexity hide karte hain
- Kaise strategy pattern se algorithms implement karte hain

---

## 📝 Requirements Analysis

### Functional Requirements

```
1. ✅ User can PLAY and PAUSE songs
2. ✅ User can CREATE playlists
3. ✅ User can ADD songs to playlists
4. ✅ User can PLAY entire playlist with different strategies:
   - Sequential (one after another)
   - Random (shuffle mode)
   - Custom (user-defined queue)
   - Extensible for future strategies
5. ✅ App should support MULTIPLE output devices:
   - Bluetooth Speaker
   - Wired Speaker
   - Headphones
   - Extensible for future devices
```

### Non-Functional Requirements

```
1. ✅ Easily SCALABLE - new features add karna easy ho
2. ✅ MAINTAINABLE code - clean and organized
3. ✅ EXTENSIBLE - new devices/strategies easily add ho sakein
4. ✅ Follow SOLID principles
5. ✅ Loose coupling between components
```

---

## 🎨 System Overview

### High-Level Flow

```
┌─────────────────────────────────────────────────────┐
│              Music Player Application               │
│                                                     │
│  Songs ──┐                                         │
│          │                                         │
│  Playlists ──┐                                     │
│              │                                     │
│  Output Devices (Bluetooth, Wired, Headphones)    │
│              │                                     │
│  Playing Strategies (Sequential, Random, Custom)   │
└─────────────────────────────────────────────────────┘
```

### Component Interaction

```
User
  │
  ▼
MusicPlayerApp (Entry Point)
  │
  ├──▶ MusicPlayerFacade (Simplifies everything)
  │     │
  │     ├──▶ AudioEngine (Plays music)
  │     │     └──▶ AudioOutputDevice (Bluetooth/Wired/Headphones)
  │     │
  │     ├──▶ DeviceManager (Manages devices)
  │     │     └──▶ DeviceFactory (Creates devices)
  │     │
  │     └──▶ PlaylistManager (Manages playlists)
  │           └──▶ PlayingStrategyManager (Sequential/Random/Custom)
  │
  └──▶ PlaylistManager (Creates playlists)
```

---

## 🎭 Design Patterns Used

Is application mein hum **5 design patterns** use kar rahe hain:

### 1. **Strategy Pattern** 🎯
- **Kahan:** Playing strategies (Sequential, Random, Custom)
- **Kyun:** Different algorithms for playing songs

### 2. **Singleton Pattern** 🔒
- **Kahan:** All managers (DeviceManager, PlaylistManager, PlayingStrategyManager)
- **Kyun:** Single instance throughout application

### 3. **Adapter Pattern** 🔌
- **Kahan:** Device adapters (BluetoothSpeakerAdapter, WiredSpeakerAdapter)
- **Kyun:** Third-party APIs ko integrate karne ke liye

### 4. **Factory Pattern** 🏭
- **Kahan:** DeviceFactory
- **Kyun:** Device creation logic centralize karne ke liye

### 5. **Facade Pattern** 🎭
- **Kahan:** MusicPlayerFacade
- **Kyun:** Complex subsystems ko simplify karne ke liye

---

## 🏗️ Bottom-Up Design Approach

Hum **bottom-up approach** follow karenge:

1. **Smallest objects** pehle banate hain (Song)
2. Phir **medium objects** (Playlist, Devices)
3. Phir **managers** (PlaylistManager, DeviceManager)
4. Finally **facade** (MusicPlayerFacade)
5. Last mein **entry point** (MusicPlayerApp)

**Benefit:** Chhote components pehle test kar sakte hain, phir unhe integrate karte hain.

---

## 🎵 Core Components

### 1. Song Model

**Purpose:** Represents a single song

```
┌─────────────────┐
│      Song       │
├─────────────────┤
│ - title: String │
│ - artist: String│
│ - path: String  │
├─────────────────┤
│ + getters()     │
│ + setters()     │
└─────────────────┘
```

**Implementation:**

```java
// Song.java
public class Song {
    private String title;
    private String artist;
    private String filePath;
    
    public Song(String title, String artist, String filePath) {
        this.title = title;
        this.artist = artist;
        this.filePath = filePath;
    }
    
    // Getters and Setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getArtist() { return artist; }
    public void setArtist(String artist) { this.artist = artist; }
    
    public String getFilePath() { return filePath; }
    public void setFilePath(String filePath) { this.filePath = filePath; }
}
```

**Key Points:**
- Simple POJO class
- Contains basic song information
- File path indicates where song is stored (database/filesystem)

---

### 2. Playlist Model

**Purpose:** Represents a collection of songs

```
┌──────────────────────┐
│      Playlist        │
├──────────────────────┤
│ - name: String       │
│ - songs: List<Song>  │
├──────────────────────┤
│ + addSong(Song)      │
│ + getSongs()         │
│ + getName()          │
└──────────────────────┘
         │
         │ HAS-A (1-to-many)
         ▼
    ┌────────┐
    │  Song  │
    └────────┘
```

**Implementation:**

```java
// Playlist.java
import java.util.ArrayList;
import java.util.List;

public class Playlist {
    private String name;
    private List<Song> songs;
    
    public Playlist(String name) {
        this.name = name;
        this.songs = new ArrayList<>();
    }
    
    public void addSong(Song song) {
        songs.add(song);
        System.out.println("Added '" + song.getTitle() + "' to playlist '" + name + "'");
    }
    
    public List<Song> getSongs() {
        return songs;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

**Key Points:**
- Contains list of songs
- Simple association with Song (not composition)
- Songs can exist without playlist

---

## 🔌 Device Management System

### Architecture Overview

```
Third-Party APIs (External)
    │
    ├── BluetoothSpeakerAPI
    ├── WiredSpeakerAPI
    └── HeadphonesAPI
         │
         │ (Adapted by)
         ▼
    Adapters
    │
    ├── BluetoothSpeakerAdapter
    ├── WiredSpeakerAdapter
    └── HeadphonesAdapter
         │
         │ (Implements)
         ▼
    IAudioOutputDevice (Interface)
         │
         │ (Used by)
         ▼
    AudioEngine
         │
         │ (Managed by)
         ▼
    DeviceManager (Singleton)
         │
         │ (Uses)
         ▼
    DeviceFactory
```

### Step 1: Audio Output Device Interface

```java
// IAudioOutputDevice.java
public interface IAudioOutputDevice {
    void playAudio(Song song);
}
```

**Purpose:** Common interface for all output devices

---

### Step 2: Third-Party APIs (External)

Ye third-party classes hain jo hume diye gaye hain:

```java
// BluetoothSpeakerAPI.java (External - Third Party)
public class BluetoothSpeakerAPI {
    public void playSoundViaBluetooth(String data) {
        System.out.println("🔊 Playing via Bluetooth Speaker: " + data);
    }
}

// WiredSpeakerAPI.java (External - Third Party)
public class WiredSpeakerAPI {
    public void playSound(String data) {
        System.out.println("🔊 Playing via Wired Speaker: " + data);
    }
}

// HeadphonesAPI.java (External - Third Party)
public class HeadphonesAPI {
    public void playSoundViaJack(String data) {
        System.out.println("🎧 Playing via Headphones: " + data);
    }
}
```

**Problem:** 
- Different method names (`playSoundViaBluetooth`, `playSound`, `playSoundViaJack`)
- Our interface expects `playAudio(Song)`
- **Solution:** Adapter Pattern!

---

### Step 3: Device Adapters

**Adapter Pattern ka use karke third-party APIs ko integrate karte hain:**

```java
// BluetoothSpeakerAdapter.java
public class BluetoothSpeakerAdapter implements IAudioOutputDevice {
    private BluetoothSpeakerAPI bluetoothSpeaker;
    
    public BluetoothSpeakerAdapter() {
        this.bluetoothSpeaker = new BluetoothSpeakerAPI();
    }
    
    @Override
    public void playAudio(Song song) {
        // Adapt: Convert Song to String data
        String data = "Song: " + song.getTitle() + " by " + song.getArtist();
        bluetoothSpeaker.playSoundViaBluetooth(data);
    }
}

// WiredSpeakerAdapter.java
public class WiredSpeakerAdapter implements IAudioOutputDevice {
    private WiredSpeakerAPI wiredSpeaker;
    
    public WiredSpeakerAdapter() {
        this.wiredSpeaker = new WiredSpeakerAPI();
    }
    
    @Override
    public void playAudio(Song song) {
        String data = "Song: " + song.getTitle() + " by " + song.getArtist();
        wiredSpeaker.playSound(data);
    }
}

// HeadphonesAdapter.java
public class HeadphonesAdapter implements IAudioOutputDevice {
    private HeadphonesAPI headphones;
    
    public HeadphonesAdapter() {
        this.headphones = new HeadphonesAPI();
    }
    
    @Override
    public void playAudio(Song song) {
        String data = "Song: " + song.getTitle() + " by " + song.getArtist();
        headphones.playSoundViaJack(data);
    }
}
```

**Relationship:**
- Adapter **IS-A** IAudioOutputDevice (implements)
- Adapter **HAS-A** Third-Party API (composition)

---

### Step 4: Device Type Enum

```java
// DeviceType.java (Enum)
public enum DeviceType {
    BLUETOOTH,
    WIRED,
    HEADPHONES
}
```

**Purpose:** Type-safe way to specify device types (better than strings)

---

### Step 5: Device Factory

**Factory Pattern se devices create karte hain:**

```java
// DeviceFactory.java
public class DeviceFactory {
    
    public IAudioOutputDevice createDevice(DeviceType deviceType) {
        switch (deviceType) {
            case BLUETOOTH:
                return new BluetoothSpeakerAdapter();
            case WIRED:
                return new WiredSpeakerAdapter();
            case HEADPHONES:
                return new HeadphonesAdapter();
            default:
                throw new IllegalArgumentException("Unknown device type: " + deviceType);
        }
    }
}
```

**Benefits:**
- Centralized creation logic
- Easy to add new devices
- Follows Open/Closed Principle

---

### Step 6: Device Manager (Singleton)

**Manager ka kaam hai devices ko manage karna:**

```java
// DeviceManager.java (Singleton)
public class DeviceManager {
    private static DeviceManager instance;
    private IAudioOutputDevice currentDevice;
    private DeviceFactory deviceFactory;
    
    // Private constructor for Singleton
    private DeviceManager() {
        this.deviceFactory = new DeviceFactory();
    }
    
    // Singleton getInstance
    public static DeviceManager getInstance() {
        if (instance == null) {
            instance = new DeviceManager();
        }
        return instance;
    }
    
    // Connect to a device
    public void connect(DeviceType deviceType) {
        currentDevice = deviceFactory.createDevice(deviceType);
        System.out.println("✅ Connected to " + deviceType + " device");
    }
    
    // Get current device
    public IAudioOutputDevice getDevice() {
        if (currentDevice == null) {
            throw new IllegalStateException("No device connected! Please connect a device first.");
        }
        return currentDevice;
    }
}
```

**Key Points:**
- **Singleton:** Only one instance throughout app
- **Pure Composition:** Device cannot exist without manager
- **Factory:** Uses DeviceFactory to create devices

---

### Step 7: Audio Engine

**Audio Engine ka kaam hai actually music play karna:**

```java
// AudioEngine.java
public class AudioEngine {
    private Song currentSong;
    private boolean isSongPaused;
    
    public AudioEngine() {
        this.isSongPaused = false;
    }
    
    // Play a song on given device
    public void play(IAudioOutputDevice device, Song song) {
        if (song == null) {
            System.out.println("❌ No song to play!");
            return;
        }
        
        // If same song is paused, resume it
        if (currentSong != null && currentSong.equals(song) && isSongPaused) {
            System.out.println("▶️  Resuming: " + song.getTitle());
            isSongPaused = false;
        }
        
        // Play the song
        currentSong = song;
        device.playAudio(song);
    }
    
    // Pause current song
    public void pause() {
        if (currentSong == null) {
            System.out.println("❌ No song is playing!");
            return;
        }
        
        if (isSongPaused) {
            System.out.println("⏸️  Song is already paused!");
            return;
        }
        
        isSongPaused = true;
        System.out.println("⏸️  Paused: " + currentSong.getTitle());
    }
    
    // Get current song title
    public String getCurrentSongTitle() {
        return currentSong != null ? currentSong.getTitle() : "No song playing";
    }
    
    // Check if song is paused
    public boolean isSongPaused() {
        return isSongPaused;
    }
}
```

**Responsibilities:**
- Tracks current song
- Handles play/pause logic
- Interacts with IAudioOutputDevice

---

## 📚 Playlist Management System

### Architecture

```
PlaylistManager (Singleton)
    │
    ├── Creates/Manages Playlists
    ├── Adds songs to playlists
    └── Retrieves playlists by name
         │
         │ (Pure Composition)
         ▼
    Playlist
         │
         │ (Simple Association)
         ▼
    Song
```

### Playlist Manager (Singleton)

```java
// PlaylistManager.java (Singleton)
import java.util.HashMap;
import java.util.Map;

public class PlaylistManager {
    private static PlaylistManager instance;
    private Map<String, Playlist> playlists; // name -> Playlist
    
    // Private constructor
    private PlaylistManager() {
        this.playlists = new HashMap<>();
    }
    
    // Singleton getInstance
    public static PlaylistManager getInstance() {
        if (instance == null) {
            instance = new PlaylistManager();
        }
        return instance;
    }
    
    // Create a new playlist
    public void createPlaylist(String name) {
        if (playlists.containsKey(name)) {
            System.out.println("⚠️  Playlist '" + name + "' already exists!");
            return;
        }
        
        Playlist playlist = new Playlist(name);
        playlists.put(name, playlist);
        System.out.println("✅ Created playlist: " + name);
    }
    
    // Add song to playlist
    public void addSongToPlaylist(String playlistName, Song song) {
        Playlist playlist = playlists.get(playlistName);
        if (playlist == null) {
            System.out.println("❌ Playlist '" + playlistName + "' not found!");
            return;
        }
        
        playlist.addSong(song);
    }
    
    // Get playlist by name
    public Playlist getPlaylist(String name) {
        Playlist playlist = playlists.get(name);
        if (playlist == null) {
            System.out.println("❌ Playlist '" + name + "' not found!");
        }
        return playlist;
    }
}
```

**Key Design Decisions:**

1. **HashMap instead of List:**
   - O(1) lookup by name
   - Better performance for large number of playlists

2. **Singleton Pattern:**
   - Only one playlist manager in entire app
   - Centralized playlist management

3. **Pure Composition:**
   - Playlist cannot exist without manager
   - Manager creates and owns playlists

---

## 🎮 Playing Strategy System

### Strategy Pattern Implementation

```
IPlayingStrategy (Interface)
    │
    ├── setPlaylist(Playlist)
    ├── next(): Song
    ├── previous(): Song
    ├── hasNext(): boolean
    ├── hasPrevious(): boolean
    └── addNextSong(Song)  // Empty by default
         │
         │ (Implemented by)
         ├──────────┬──────────┬──────────
         │          │          │
         ▼          ▼          ▼
  Sequential   Random    Custom
   Strategy    Strategy  Strategy
```

### Step 1: Strategy Type Enum

```java
// StrategyType.java (Enum)
public enum StrategyType {
    SEQUENTIAL,
    RANDOM,
    CUSTOM
}
```

---

### Step 2: Playing Strategy Interface

```java
// IPlayingStrategy.java
public interface IPlayingStrategy {
    void setPlaylist(Playlist playlist);
    Song next();
    Song previous();
    boolean hasNext();
    boolean hasPrevious();
    
    // Default empty implementation (only CustomStrategy will override)
    default void addNextSong(Song song) {
        // Empty - only custom strategy uses this
    }
}
```

**Why `addNextSong` is default empty?**
- Only `CustomStrategy` needs it
- Other strategies don't need to implement it
- Avoids Interface Segregation Principle violation
- Better than forcing all strategies to implement unused method

---

### Step 3: Sequential Playing Strategy

**Plays songs one after another in order:**

```java
// SequentialPlayingStrategy.java
import java.util.List;

public class SequentialPlayingStrategy implements IPlayingStrategy {
    private Playlist playlist;
    private int currentIndex;
    
    public SequentialPlayingStrategy() {
        this.currentIndex = -1;
    }
    
    @Override
    public void setPlaylist(Playlist playlist) {
        this.playlist = playlist;
        this.currentIndex = -1;
    }
    
    @Override
    public Song next() {
        if (!hasNext()) {
            System.out.println("❌ No more songs in playlist!");
            return null;
        }
        
        currentIndex++;
        return playlist.getSongs().get(currentIndex);
    }
    
    @Override
    public Song previous() {
        if (!hasPrevious()) {
            System.out.println("❌ Already at first song!");
            return null;
        }
        
        currentIndex--;
        return playlist.getSongs().get(currentIndex);
    }
    
    @Override
    public boolean hasNext() {
        if (playlist == null) return false;
        return currentIndex < playlist.getSongs().size() - 1;
    }
    
    @Override
    public boolean hasPrevious() {
        return currentIndex > 0;
    }
}
```

**Algorithm:**
- Maintains `currentIndex`
- `next()`: Increment index, return song
- `previous()`: Decrement index, return song
- Simple and straightforward

---

### Step 4: Random Playing Strategy

**Plays songs in random order:**

```java
// RandomPlayingStrategy.java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class RandomPlayingStrategy implements IPlayingStrategy {
    private Playlist playlist;
    private List<Song> shuffledSongs;
    private int currentIndex;
    
    public RandomPlayingStrategy() {
        this.currentIndex = -1;
    }
    
    @Override
    public void setPlaylist(Playlist playlist) {
        this.playlist = playlist;
        this.currentIndex = -1;
        
        // Create shuffled copy
        shuffledSongs = new ArrayList<>(playlist.getSongs());
        Collections.shuffle(shuffledSongs);
    }
    
    @Override
    public Song next() {
        if (!hasNext()) {
            System.out.println("❌ No more songs in playlist!");
            return null;
        }
        
        currentIndex++;
        return shuffledSongs.get(currentIndex);
    }
    
    @Override
    public Song previous() {
        if (!hasPrevious()) {
            System.out.println("❌ Already at first song!");
            return null;
        }
        
        currentIndex--;
        return shuffledSongs.get(currentIndex);
    }
    
    @Override
    public boolean hasNext() {
        if (shuffledSongs == null) return false;
        return currentIndex < shuffledSongs.size() - 1;
    }
    
    @Override
    public boolean hasPrevious() {
        return currentIndex > 0;
    }
}
```

**Algorithm:**
- Creates shuffled copy of songs
- Uses `Collections.shuffle()`
- Rest is same as sequential

---

### Step 5: Custom Playing Strategy

**User defines which song plays next:**

```java
// CustomPlayingStrategy.java
import java.util.LinkedList;
import java.util.Queue;

public class CustomPlayingStrategy implements IPlayingStrategy {
    private Playlist playlist;
    private Queue<Song> customQueue;
    private Song currentSong;
    
    public CustomPlayingStrategy() {
        this.customQueue = new LinkedList<>();
    }
    
    @Override
    public void setPlaylist(Playlist playlist) {
        this.playlist = playlist;
    }
    
    @Override
    public void addNextSong(Song song) {
        customQueue.offer(song);
        System.out.println("➕ Added to queue: " + song.getTitle());
    }
    
    @Override
    public Song next() {
        if (!hasNext()) {
            System.out.println("❌ Queue is empty!");
            return null;
        }
        
        currentSong = customQueue.poll();
        return currentSong;
    }
    
    @Override
    public Song previous() {
        // In custom mode, previous returns current song again
        return currentSong;
    }
    
    @Override
    public boolean hasNext() {
        return !customQueue.isEmpty();
    }
    
    @Override
    public boolean hasPrevious() {
        return currentSong != null;
    }
}
```

**Algorithm:**
- Uses Queue data structure
- User adds songs via `addNextSong()`
- `next()` polls from queue
- `previous()` returns current song (no real previous in queue)

---

### Step 6: Playing Strategy Manager (Singleton)

```java
// PlayingStrategyManager.java (Singleton)
public class PlayingStrategyManager {
    private static PlayingStrategyManager instance;
    
    private SequentialPlayingStrategy sequentialStrategy;
    private RandomPlayingStrategy randomStrategy;
    private CustomPlayingStrategy customStrategy;
    
    // Private constructor
    private PlayingStrategyManager() {
        this.sequentialStrategy = new SequentialPlayingStrategy();
        this.randomStrategy = new RandomPlayingStrategy();
        this.customStrategy = new CustomPlayingStrategy();
    }
    
    // Singleton getInstance
    public static PlayingStrategyManager getInstance() {
        if (instance == null) {
            instance = new PlayingStrategyManager();
        }
        return instance;
    }
    
    // Get strategy based on type
    public IPlayingStrategy getStrategy(StrategyType strategyType) {
        switch (strategyType) {
            case SEQUENTIAL:
                return sequentialStrategy;
            case RANDOM:
                return randomStrategy;
            case CUSTOM:
                return customStrategy;
            default:
                throw new IllegalArgumentException("Unknown strategy type: " + strategyType);
        }
    }
}
```

**Key Points:**
- Pre-creates all strategies (reuses them)
- Returns appropriate strategy based on type
- Singleton pattern

---

## 🎭 Facade Layer

### MusicPlayerFacade

**Facade pattern se saari complexity hide karte hain:**

```java
// MusicPlayerFacade.java
public class MusicPlayerFacade {
    private AudioEngine audioEngine;
    private DeviceManager deviceManager;
    private PlaylistManager playlistManager;
    private PlayingStrategyManager strategyManager;
    
    private IPlayingStrategy currentStrategy;
    private Playlist currentPlaylist;
    
    public MusicPlayerFacade() {
        this.audioEngine = new AudioEngine();
        this.deviceManager = DeviceManager.getInstance();
        this.playlistManager = PlaylistManager.getInstance();
        this.strategyManager = PlayingStrategyManager.getInstance();
    }
    
    // ========== Device Management ==========
    
    public void connectDevice(DeviceType deviceType) {
        deviceManager.connect(deviceType);
    }
    
    // ========== Single Song Operations ==========
    
    public void play(Song song) {
        IAudioOutputDevice device = deviceManager.getDevice();
        audioEngine.play(device, song);
    }
    
    public void pause() {
        audioEngine.pause();
    }
    
    // ========== Playlist Operations ==========
    
    public void loadPlaylist(String playlistName) {
        currentPlaylist = playlistManager.getPlaylist(playlistName);
        if (currentPlaylist != null && currentStrategy != null) {
            currentStrategy.setPlaylist(currentPlaylist);
        }
    }
    
    public void setStrategy(StrategyType strategyType) {
        currentStrategy = strategyManager.getStrategy(strategyType);
        if (currentPlaylist != null) {
            currentStrategy.setPlaylist(currentPlaylist);
        }
        System.out.println("🎵 Strategy set to: " + strategyType);
    }
    
    public void playAll(String playlistName) {
        // Load playlist
        loadPlaylist(playlistName);
        
        if (currentPlaylist == null || currentStrategy == null) {
            System.out.println("❌ Playlist or strategy not set!");
            return;
        }
        
        // Get device
        IAudioOutputDevice device = deviceManager.getDevice();
        
        // Play all songs using strategy
        while (currentStrategy.hasNext()) {
            Song song = currentStrategy.next();
            audioEngine.play(device, song);
        }
    }
    
    public void playNext() {
        if (currentStrategy == null) {
            System.out.println("❌ No strategy set!");
            return;
        }
        
        Song nextSong = currentStrategy.next();
        if (nextSong != null) {
            IAudioOutputDevice device = deviceManager.getDevice();
            audioEngine.play(device, nextSong);
        }
    }
    
    public void playPrevious() {
        if (currentStrategy == null) {
            System.out.println("❌ No strategy set!");
            return;
        }
        
        Song previousSong = currentStrategy.previous();
        if (previousSong != null) {
            IAudioOutputDevice device = deviceManager.getDevice();
            audioEngine.play(device, previousSong);
        }
    }
    
    public void addToQueue(Song song) {
        if (currentStrategy != null) {
            currentStrategy.addNextSong(song);
        }
    }
}
```

**Facade Benefits:**
- Client doesn't know about AudioEngine, DeviceManager, etc.
- Simple interface: `play()`, `pause()`, `playAll()`
- Hides complex subsystem interactions
- Single point of control

---

## 🚀 Entry Point: MusicPlayerApp

**Single entry point for entire application:**

```java
// MusicPlayerApp.java
import java.util.ArrayList;
import java.util.List;

public class MusicPlayerApp {
    private List<Song> allSongs;
    private MusicPlayerFacade facade;
    private PlaylistManager playlistManager;
    
    public MusicPlayerApp() {
        this.allSongs = new ArrayList<>();
        this.facade = new MusicPlayerFacade();
        this.playlistManager = PlaylistManager.getInstance();
    }
    
    // Create a song
    public void createSong(String title, String artist, String filePath) {
        Song song = new Song(title, artist, filePath);
        allSongs.add(song);
        System.out.println("✅ Created song: " + title);
    }
    
    // Create a playlist
    public void createPlaylist(String name) {
        playlistManager.createPlaylist(name);
    }
    
    // Add song to playlist
    public void addSongToPlaylist(String playlistName, Song song) {
        playlistManager.addSongToPlaylist(playlistName, song);
    }
    
    // Connect device
    public void connectDevice(DeviceType deviceType) {
        facade.connectDevice(deviceType);
    }
    
    // Play single song
    public void playSong(Song song) {
        facade.play(song);
    }
    
    // Pause song
    public void pauseSong() {
        facade.pause();
    }
    
    // Set playing strategy
    public void setPlayingStrategy(StrategyType strategyType) {
        facade.setStrategy(strategyType);
    }
    
    // Play entire playlist
    public void playPlaylist(String playlistName) {
        facade.playAll(playlistName);
    }
    
    // Play next song
    public void playNext() {
        facade.playNext();
    }
    
    // Play previous song
    public void playPrevious() {
        facade.playPrevious();
    }
    
    // Get all songs
    public List<Song> getAllSongs() {
        return allSongs;
    }
}
```

**Responsibilities:**
- Entry point for user
- Manages all songs
- Delegates to facade for complex operations
- Delegates to PlaylistManager for playlist creation

---

## 📊 Complete UML Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    MusicPlayerApp (Entry Point)                 │
│  - allSongs: List<Song>                                         │
│  - facade: MusicPlayerFacade                                    │
│  - playlistManager: PlaylistManager                             │
│  + createSong(), createPlaylist(), playSong(), etc.             │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ├──▶ MusicPlayerFacade
                         │     │
                         │     ├──▶ AudioEngine
                         │     │     └──▶ IAudioOutputDevice
                         │     │           │
                         │     │           ├── BluetoothSpeakerAdapter
                         │     │           ├── WiredSpeakerAdapter
                         │     │           └── HeadphonesAdapter
                         │     │                 │
                         │     │                 └──▶ Third-Party APIs
                         │     │
                         │     ├──▶ DeviceManager (Singleton)
                         │     │     └──▶ DeviceFactory
                         │     │
                         │     └──▶ PlayingStrategyManager (Singleton)
                         │           └──▶ IPlayingStrategy
                         │                 │
                         │                 ├── SequentialStrategy
                         │                 ├── RandomStrategy
                         │                 └── CustomStrategy
                         │
                         └──▶ PlaylistManager (Singleton)
                               └──▶ Playlist
                                     └──▶ Song
```

---

## 💻 Implementation

### Main Class - Demo

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println("🎵 ===== Music Player Application ===== 🎵\n");
        
        // Create app
        MusicPlayerApp app = new MusicPlayerApp();
        
        // Step 1: Create songs
        System.out.println("📀 Creating songs...");
        app.createSong("Shape of You", "Ed Sheeran", "/music/shape.mp3");
        app.createSong("Blinding Lights", "The Weeknd", "/music/blinding.mp3");
        app.createSong("Levitating", "Dua Lipa", "/music/levitating.mp3");
        app.createSong("Peaches", "Justin Bieber", "/music/peaches.mp3");
        
        List<Song> songs = app.getAllSongs();
        System.out.println();
        
        // Step 2: Create playlist
        System.out.println("📝 Creating playlist...");
        app.createPlaylist("My Favorites");
        app.addSongToPlaylist("My Favorites", songs.get(0));
        app.addSongToPlaylist("My Favorites", songs.get(1));
        app.addSongToPlaylist("My Favorites", songs.get(2));
        System.out.println();
        
        // Step 3: Connect device
        System.out.println("🔌 Connecting device...");
        app.connectDevice(DeviceType.BLUETOOTH);
        System.out.println();
        
        // Step 4: Play single song
        System.out.println("▶️  Playing single song...");
        app.playSong(songs.get(0));
        System.out.println();
        
        // Step 5: Pause song
        System.out.println("⏸️  Pausing song...");
        app.pauseSong();
        System.out.println();
        
        // Step 6: Play playlist with sequential strategy
        System.out.println("🎵 Playing playlist (Sequential)...");
        app.setPlayingStrategy(StrategyType.SEQUENTIAL);
        app.playPlaylist("My Favorites");
        System.out.println();
        
        // Step 7: Change device
        System.out.println("🔌 Changing device to Headphones...");
        app.connectDevice(DeviceType.HEADPHONES);
        System.out.println();
        
        // Step 8: Play with random strategy
        System.out.println("🎵 Playing playlist (Random)...");
        app.setPlayingStrategy(StrategyType.RANDOM);
        app.playPlaylist("My Favorites");
        System.out.println();
        
        System.out.println("🎵 ===== Demo Complete ===== 🎵");
    }
}
```

**Output:**
```
🎵 ===== Music Player Application ===== 🎵

📀 Creating songs...
✅ Created song: Shape of You
✅ Created song: Blinding Lights
✅ Created song: Levitating
✅ Created song: Peaches

📝 Creating playlist...
✅ Created playlist: My Favorites
Added 'Shape of You' to playlist 'My Favorites'
Added 'Blinding Lights' to playlist 'My Favorites'
Added 'Levitating' to playlist 'My Favorites'

🔌 Connecting device...
✅ Connected to BLUETOOTH device

▶️  Playing single song...
🔊 Playing via Bluetooth Speaker: Song: Shape of You by Ed Sheeran

⏸️  Pausing song...
⏸️  Paused: Shape of You

🎵 Playing playlist (Sequential)...
🎵 Strategy set to: SEQUENTIAL
🔊 Playing via Bluetooth Speaker: Song: Shape of You by Ed Sheeran
🔊 Playing via Bluetooth Speaker: Song: Blinding Lights by The Weeknd
🔊 Playing via Bluetooth Speaker: Song: Levitating by Dua Lipa

🔌 Changing device to Headphones...
✅ Connected to HEADPHONES device

🎵 Playing playlist (Random)...
🎵 Strategy set to: RANDOM
🎧 Playing via Headphones: Song: Levitating by Dua Lipa
🎧 Playing via Headphones: Song: Shape of You by Ed Sheeran
🎧 Playing via Headphones: Song: Blinding Lights by The Weeknd

🎵 ===== Demo Complete ===== 🎵
```

---

## 🔑 Key Design Decisions

### 1. Why Managers are Singleton?

**Reason:**
- Single source of truth
- Centralized management
- Avoid duplicate instances
- Memory efficient

**Example:**
```java
// Only one DeviceManager in entire app
DeviceManager dm1 = DeviceManager.getInstance();
DeviceManager dm2 = DeviceManager.getInstance();
// dm1 == dm2 (same instance)
```

---

### 2. Why HashMap in PlaylistManager?

**Reason:**
- O(1) lookup by name
- Better than List (O(n) lookup)
- Efficient for large number of playlists

**Comparison:**
```java
// List approach - O(n)
for (Playlist p : playlists) {
    if (p.getName().equals(name)) return p;
}

// HashMap approach - O(1)
return playlists.get(name);
```

---

### 3. Why Pure Composition for DeviceManager?

**Reason:**
- Device should not exist without manager
- Manager creates and owns devices
- Tight lifecycle coupling

**Diagram:**
```
DeviceManager ◆────▶ IAudioOutputDevice
(Pure Composition - filled diamond)
```

---

### 4. Why Simple Association for Playlist-Song?

**Reason:**
- Song can exist without playlist
- Song can be in multiple playlists
- Loose coupling

**Diagram:**
```
Playlist ◇────▶ Song
(Simple Association - empty diamond)
```

---

### 5. Why Default Method in IPlayingStrategy?

**Problem:**
```java
// If we make addNextSong() abstract:
public interface IPlayingStrategy {
    void addNextSong(Song song); // All must implement
}

// Sequential doesn't need it!
public class SequentialStrategy implements IPlayingStrategy {
    public void addNextSong(Song song) {
        // Empty - waste!
    }
}
```

**Solution:**
```java
// Default empty implementation
public interface IPlayingStrategy {
    default void addNextSong(Song song) {
        // Empty - only Custom overrides
    }
}
```

**Benefits:**
- Avoids Interface Segregation Principle violation
- Only CustomStrategy overrides it
- Others get empty implementation for free

---

### 6. Why Facade Pattern?

**Without Facade:**
```java
// Client has to know everything!
DeviceManager dm = DeviceManager.getInstance();
dm.connect(DeviceType.BLUETOOTH);
IAudioOutputDevice device = dm.getDevice();
AudioEngine engine = new AudioEngine();
engine.play(device, song);
```

**With Facade:**
```java
// Simple!
facade.connectDevice(DeviceType.BLUETOOTH);
facade.play(song);
```

**Benefits:**
- Hides complexity
- Simple interface
- Loose coupling

---

## 🌍 Real-World Applications

### 1. Spotify / Apple Music

**Similar Design:**
- Multiple output devices (Bluetooth, AirPlay, etc.)
- Playlists with different play modes
- Strategy pattern for shuffle/repeat
- Facade for simple user interface

---

### 2. Video Players (VLC, YouTube)

**Similar Concepts:**
- Different output devices (HDMI, Bluetooth, etc.)
- Playlists of videos
- Sequential/Random/Custom playback
- Adapter for different video formats

---

### 3. Smart Home Systems

**Similar Architecture:**
- Multiple devices (lights, fans, AC)
- Scenes (like playlists)
- Different execution strategies
- Facade to simplify control

---

## ❓ Interview Q&A

### Q1: Explain the overall architecture of Music Player Application.

**Answer:**

**Architecture Overview:**
```
Entry Point (MusicPlayerApp)
    ↓
Facade Layer (MusicPlayerFacade)
    ↓
Core Components:
    - AudioEngine (plays music)
    - DeviceManager (manages devices)
    - PlaylistManager (manages playlists)
    - PlayingStrategyManager (manages strategies)
    ↓
Lower Level:
    - Devices (Adapters + Third-party APIs)
    - Strategies (Sequential/Random/Custom)
    - Models (Song, Playlist)
```

**Design Patterns Used:**
1. **Facade:** MusicPlayerFacade simplifies complex subsystems
2. **Adapter:** Device adapters integrate third-party APIs
3. **Strategy:** Different playing algorithms
4. **Singleton:** All managers
5. **Factory:** DeviceFactory creates devices

**Flow Example:**
```
User → MusicPlayerApp → Facade → AudioEngine → Device → Speaker
```

---

### Q2: Why did you use Adapter Pattern for devices?

**Answer:**

**Problem:**
Third-party APIs have different interfaces:
```java
BluetoothSpeakerAPI.playSoundViaBluetooth(String)
WiredSpeakerAPI.playSound(String)
HeadphonesAPI.playSoundViaJack(String)
```

Our interface expects:
```java
IAudioOutputDevice.playAudio(Song)
```

**Solution: Adapter Pattern**
```java
public class BluetoothSpeakerAdapter implements IAudioOutputDevice {
    private BluetoothSpeakerAPI api;
    
    public void playAudio(Song song) {
        String data = convert(song);
        api.playSoundViaBluetooth(data); // Adapt!
    }
}
```

**Benefits:**
- Uniform interface for all devices
- Easy to add new devices
- Follows Open/Closed Principle
- Client doesn't know about third-party APIs

---

### Q3: Why did you use Strategy Pattern for playing modes?

**Answer:**

**Problem:**
Different algorithms for playing songs:
- Sequential: Play one after another
- Random: Shuffle and play
- Custom: User-defined queue

**Without Strategy (Bad):**
```java
public void playAll(String mode) {
    if (mode.equals("sequential")) {
        // Sequential logic
    } else if (mode.equals("random")) {
        // Random logic
    } else if (mode.equals("custom")) {
        // Custom logic
    }
}
```

**Problems:**
- Violates Open/Closed Principle
- Hard to add new strategies
- Complex if-else chains

**With Strategy (Good):**
```java
IPlayingStrategy strategy = strategyManager.getStrategy(type);
while (strategy.hasNext()) {
    Song song = strategy.next();
    play(song);
}
```

**Benefits:**
- Easy to add new strategies
- Follows Open/Closed Principle
- Clean separation of algorithms
- Runtime strategy switching

---

### Q4: Why are all managers Singleton?

**Answer:**

**Reasons:**

1. **Single Source of Truth:**
   - Only one DeviceManager should manage all devices
   - Only one PlaylistManager should manage all playlists
   - Avoids inconsistencies

2. **Resource Management:**
   - Managers hold state (connected device, playlists)
   - Multiple instances would duplicate state
   - Memory inefficient

3. **Centralized Control:**
   - Easy to access from anywhere
   - No need to pass manager references everywhere

**Example:**
```java
// Anywhere in code
DeviceManager dm = DeviceManager.getInstance();
dm.connect(DeviceType.BLUETOOTH);

// Later, somewhere else
DeviceManager dm2 = DeviceManager.getInstance();
IAudioOutputDevice device = dm2.getDevice(); // Same device!
```

**Trade-off:**
- Global state (can be problematic in testing)
- But benefits outweigh in this use case

---

### Q5: Explain the flow when user plays a playlist.

**Answer:**

**Step-by-Step Flow:**

```
1. User calls: app.playPlaylist("My Favorites")
   ↓
2. MusicPlayerApp delegates to Facade
   facade.playAll("My Favorites")
   ↓
3. Facade loads playlist
   playlist = playlistManager.getPlaylist("My Favorites")
   ↓
4. Facade sets playlist in strategy
   currentStrategy.setPlaylist(playlist)
   ↓
5. Facade gets device
   device = deviceManager.getDevice()
   ↓
6. Facade loops through songs
   while (strategy.hasNext()) {
       song = strategy.next()  // Strategy decides which song
       audioEngine.play(device, song)  // Engine plays it
   }
   ↓
7. AudioEngine plays each song
   device.playAudio(song)  // Device adapter
   ↓
8. Adapter calls third-party API
   bluetoothAPI.playSoundViaBluetooth(data)
   ↓
9. Music plays! 🎵
```

**Key Points:**
- Facade orchestrates everything
- Strategy decides song order
- AudioEngine handles playback
- Adapter integrates with hardware

---

### Q6: Why did you use Facade Pattern?

**Answer:**

**Problem: Complex Subsystem**

Without Facade, client needs to know:
```java
// Too complex!
DeviceManager dm = DeviceManager.getInstance();
dm.connect(DeviceType.BLUETOOTH);

PlaylistManager pm = PlaylistManager.getInstance();
Playlist playlist = pm.getPlaylist("Favorites");

PlayingStrategyManager sm = PlayingStrategyManager.getInstance();
IPlayingStrategy strategy = sm.getStrategy(StrategyType.SEQUENTIAL);
strategy.setPlaylist(playlist);

IAudioOutputDevice device = dm.getDevice();
AudioEngine engine = new AudioEngine();

while (strategy.hasNext()) {
    Song song = strategy.next();
    engine.play(device, song);
}
```

**With Facade:**
```java
// Simple!
facade.connectDevice(DeviceType.BLUETOOTH);
facade.setStrategy(StrategyType.SEQUENTIAL);
facade.playAll("Favorites");
```

**Benefits:**
- **Simplifies:** Complex subsystem hidden
- **Decouples:** Client doesn't know about managers
- **Maintainable:** Changes in subsystem don't affect client
- **Single Point:** One place to control everything

---

### Q7: How would you add a new device (e.g., WiFi Speaker)?

**Answer:**

**Steps:**

1. **Create Third-Party API:**
```java
public class WiFiSpeakerAPI {
    public void streamAudio(String data) {
        System.out.println("📡 Streaming via WiFi: " + data);
    }
}
```

2. **Create Adapter:**
```java
public class WiFiSpeakerAdapter implements IAudioOutputDevice {
    private WiFiSpeakerAPI wifiSpeaker;
    
    public WiFiSpeakerAdapter() {
        this.wifiSpeaker = new WiFiSpeakerAPI();
    }
    
    @Override
    public void playAudio(Song song) {
        String data = "Song: " + song.getTitle();
        wifiSpeaker.streamAudio(data);
    }
}
```

3. **Add to DeviceType Enum:**
```java
public enum DeviceType {
    BLUETOOTH,
    WIRED,
    HEADPHONES,
    WIFI  // New!
}
```

4. **Update DeviceFactory:**
```java
public IAudioOutputDevice createDevice(DeviceType type) {
    switch (type) {
        case BLUETOOTH: return new BluetoothSpeakerAdapter();
        case WIRED: return new WiredSpeakerAdapter();
        case HEADPHONES: return new HeadphonesAdapter();
        case WIFI: return new WiFiSpeakerAdapter();  // New!
        default: throw new IllegalArgumentException("Unknown type");
    }
}
```

**That's it! No other changes needed!**

**Why so easy?**
- Adapter Pattern makes it extensible
- Factory centralizes creation
- Follows Open/Closed Principle

---

### Q8: How would you add a new playing strategy (e.g., Most Played)?

**Answer:**

**Steps:**

1. **Create Strategy Class:**
```java
public class MostPlayedStrategy implements IPlayingStrategy {
    private Playlist playlist;
    private List<Song> sortedByPlayCount;
    private int currentIndex;
    
    @Override
    public void setPlaylist(Playlist playlist) {
        this.playlist = playlist;
        // Sort songs by play count (descending)
        sortedByPlayCount = new ArrayList<>(playlist.getSongs());
        sortedByPlayCount.sort((s1, s2) -> 
            Integer.compare(s2.getPlayCount(), s1.getPlayCount())
        );
        currentIndex = -1;
    }
    
    @Override
    public Song next() {
        if (!hasNext()) return null;
        currentIndex++;
        return sortedByPlayCount.get(currentIndex);
    }
    
    // ... implement other methods
}
```

2. **Add to StrategyType Enum:**
```java
public enum StrategyType {
    SEQUENTIAL,
    RANDOM,
    CUSTOM,
    MOST_PLAYED  // New!
}
```

3. **Update PlayingStrategyManager:**
```java
public class PlayingStrategyManager {
    private MostPlayedStrategy mostPlayedStrategy;  // New!
    
    private PlayingStrategyManager() {
        // ... existing strategies
        this.mostPlayedStrategy = new MostPlayedStrategy();
    }
    
    public IPlayingStrategy getStrategy(StrategyType type) {
        switch (type) {
            case SEQUENTIAL: return sequentialStrategy;
            case RANDOM: return randomStrategy;
            case CUSTOM: return customStrategy;
            case MOST_PLAYED: return mostPlayedStrategy;  // New!
            default: throw new IllegalArgumentException("Unknown type");
        }
    }
}
```

**That's it!**

**Why so easy?**
- Strategy Pattern makes it extensible
- Just implement interface
- Manager handles creation

---

### Q9: What SOLID principles are followed in this design?

**Answer:**

**1. Single Responsibility Principle (SRP):**
- `Song`: Only represents song data
- `AudioEngine`: Only handles playback
- `DeviceManager`: Only manages devices
- `PlaylistManager`: Only manages playlists
- Each class has ONE reason to change

**2. Open/Closed Principle (OCP):**
- **Open for extension:**
  - Add new devices without modifying existing code
  - Add new strategies without modifying existing code
- **Closed for modification:**
  - DeviceFactory handles new devices
  - StrategyManager handles new strategies

**3. Liskov Substitution Principle (LSP):**
- Any `IAudioOutputDevice` can replace another
- Any `IPlayingStrategy` can replace another
- Parent interface can be substituted by any child

**4. Interface Segregation Principle (ISP):**
- `IAudioOutputDevice`: Only one method `playAudio()`
- `IPlayingStrategy`: Only necessary methods
- `addNextSong()` has default implementation (not forced on all)

**5. Dependency Inversion Principle (DIP):**
- AudioEngine depends on `IAudioOutputDevice` (interface)
- Facade depends on `IPlayingStrategy` (interface)
- Not on concrete classes

---

### Q10: What are the advantages and disadvantages of this design?

**Answer:**

**Advantages:**

✅ **Highly Extensible:**
- Easy to add new devices
- Easy to add new strategies
- Minimal code changes

✅ **Loosely Coupled:**
- Components don't know about each other
- Changes in one don't affect others

✅ **Follows SOLID:**
- Clean, maintainable code
- Easy to test

✅ **Scalable:**
- Can handle thousands of songs/playlists
- HashMap for O(1) lookups

✅ **Real-World Ready:**
- Handles third-party integrations
- Supports multiple devices
- Flexible playback modes

**Disadvantages:**

❌ **Complexity:**
- Many classes (might be overkill for simple app)
- Steep learning curve for beginners

❌ **Over-Engineering:**
- For small app, might be too much
- Simple app doesn't need all patterns

❌ **Singleton Issues:**
- Global state (hard to test)
- Thread-safety concerns (not addressed here)

❌ **Memory:**
- Multiple managers in memory
- Strategy instances pre-created

**When to Use:**
- Large-scale applications
- Need for extensibility
- Multiple integrations
- Long-term maintenance

**When NOT to Use:**
- Simple music player
- Prototype/MVP
- Limited features

---

## 🎓 Summary

**Key Takeaways:**

1. **Bottom-Up Design:**
   - Start with smallest components (Song)
   - Build up to complex systems (Facade)

2. **Multiple Patterns:**
   - Facade: Simplify complex subsystems
   - Adapter: Integrate third-party APIs
   - Strategy: Different algorithms
   - Singleton: Single instance managers
   - Factory: Centralized creation

3. **Manager Pattern:**
   - Managers handle CRUD operations
   - Usually Singleton
   - Centralized control

4. **Composition Over Inheritance:**
   - HAS-A relationships
   - Pure composition where needed
   - Simple association where appropriate

5. **SOLID Principles:**
   - SRP: One responsibility per class
   - OCP: Open for extension, closed for modification
   - LSP: Substitutable interfaces
   - ISP: Small, focused interfaces
   - DIP: Depend on abstractions

**Interview Tips:**

- Start with requirements
- Draw diagrams first
- Explain design decisions
- Mention trade-offs
- Show extensibility
- Discuss SOLID principles

**Real-World Relevance:**

Ye design pattern sirf music player ke liye nahi hai. Isko use kar sakte ho:
- Video players
- Game engines
- Smart home systems
- Any system with multiple devices/strategies

---

**Remember:** Interview mein itna bada design nahi banana padega. Ye complete example hai taaki aap samajh sako ki real-world mein kaise design karte hain. Interview mein iska ek part bhi implement kar paye toh bahut achha hai! 🎯

---
