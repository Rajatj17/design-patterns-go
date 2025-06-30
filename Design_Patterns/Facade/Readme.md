# Design Patterns Guide: Facade Pattern

## Table of Contents
- [Design Patterns Guide: Facade Pattern](#design-patterns-guide-facade-pattern)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
    - [Key Components:](#key-components)
    - [Real-world Analogy:](#real-world-analogy)
  - [Facade Pattern Fundamentals](#facade-pattern-fundamentals)
    - [Problem It Solves:](#problem-it-solves)
    - [When to Use:](#when-to-use)
    - [Structure:](#structure)
  - [Implementation in Java](#implementation-in-java)
    - [Java Implementation - Home Theater System Example](#java-implementation---home-theater-system-example)
    - [Java Output:](#java-output)
  - [Implementation in Go](#implementation-in-go)
    - [Go Implementation - Home Theater System Example](#go-implementation---home-theater-system-example)
    - [Go Output:](#go-output)

## Overview

The **Facade Pattern** is a structural design pattern that provides a simplified interface to a complex subsystem. It defines a higher-level interface that makes the subsystem easier to use by hiding the complexities of the subsystem from clients.

### Key Components:
- **Facade**: Provides a simple interface to the complex subsystem
- **Subsystem Classes**: Implement subsystem functionality and handle work assigned by the Facade
- **Client**: Uses the Facade instead of calling subsystem objects directly

### Real-world Analogy:
Think of a car's dashboard. When you want to start the car, you simply turn the key or press a button. The dashboard (Facade) hides the complex interactions between the engine, fuel system, electrical system, and transmission. You don't need to understand the internal workings of each subsystem.

## Facade Pattern Fundamentals

### Problem It Solves:
- **Complex subsystem interactions** - Multiple classes with complex dependencies
- **Tight coupling** - Clients directly depend on many subsystem classes
- **Difficult API usage** - Subsystem APIs are hard to understand and use
- **Code duplication** - Same sequence of subsystem calls repeated across clients

### When to Use:
- You need to provide a simple interface to a complex subsystem
- You want to decouple clients from subsystem implementation details
- You need to layer your subsystems
- You want to wrap a poorly designed collection of APIs

### Structure:
```
Client
└── uses Facade

Facade
├── subsystemOperation1()
├── subsystemOperation2()
└── coordinates SubsystemA, SubsystemB, SubsystemC

SubsystemA
├── operationA1()
└── operationA2()

SubsystemB
├── operationB1()
└── operationB2()

SubsystemC
├── operationC1()
└── operationC2()
```

## Implementation in Java

### Java Implementation - Home Theater System Example

```java
// Subsystem Classes - Complex components of a home theater system

// Audio System
class AudioSystem {
    private String brand;
    private int volume;
    
    public AudioSystem(String brand) {
        this.brand = brand;
        this.volume = 0;
    }
    
    public void powerOn() {
        System.out.println(brand + " Audio System: Powering on...");
    }
    
    public void powerOff() {
        System.out.println(brand + " Audio System: Powering off...");
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println(brand + " Audio System: Volume set to " + volume);
    }
    
    public void setSurroundSound(boolean enabled) {
        System.out.println(brand + " Audio System: Surround sound " + 
                          (enabled ? "enabled" : "disabled"));
    }
    
    public void selectInput(String input) {
        System.out.println(brand + " Audio System: Input selected - " + input);
    }
}

// Video System
class VideoSystem {
    private String brand;
    private String resolution;
    
    public VideoSystem(String brand) {
        this.brand = brand;
        this.resolution = "1080p";
    }
    
    public void powerOn() {
        System.out.println(brand + " Video System: Powering on...");
    }
    
    public void powerOff() {
        System.out.println(brand + " Video System: Powering off...");
    }
    
    public void setResolution(String resolution) {
        this.resolution = resolution;
        System.out.println(brand + " Video System: Resolution set to " + resolution);
    }
    
    public void selectInput(String input) {
        System.out.println(brand + " Video System: Input selected - " + input);
    }
    
    public void adjustBrightness(int brightness) {
        System.out.println(brand + " Video System: Brightness adjusted to " + brightness);
    }
}

// Lighting System
class LightingSystem {
    private String[] zones = {"Living Room", "Kitchen", "Bedroom"};
    
    public void dimLights(int percentage) {
        System.out.println("Lighting System: Dimming all lights to " + percentage + "%");
        for (String zone : zones) {
            System.out.println("  - " + zone + " lights dimmed");
        }
    }
    
    public void turnOffLights() {
        System.out.println("Lighting System: Turning off all lights");
        for (String zone : zones) {
            System.out.println("  - " + zone + " lights off");
        }
    }
    
    public void turnOnLights() {
        System.out.println("Lighting System: Turning on all lights");
        for (String zone : zones) {
            System.out.println("  - " + zone + " lights on");
        }
    }
    
    public void setAmbientLighting() {
        System.out.println("Lighting System: Setting ambient lighting for movie watching");
    }
}

// Media Player
class MediaPlayer {
    private String brand;
    private boolean isPlaying;
    
    public MediaPlayer(String brand) {
        this.brand = brand;
        this.isPlaying = false;
    }
    
    public void powerOn() {
        System.out.println(brand + " Media Player: Powering on...");
    }
    
    public void powerOff() {
        System.out.println(brand + " Media Player: Powering off...");
    }
    
    public void loadMedia(String mediaType, String title) {
        System.out.println(brand + " Media Player: Loading " + mediaType + " - " + title);
    }
    
    public void play() {
        isPlaying = true;
        System.out.println(brand + " Media Player: Playing media...");
    }
    
    public void pause() {
        isPlaying = false;
        System.out.println(brand + " Media Player: Media paused");
    }
    
    public void stop() {
        isPlaying = false;
        System.out.println(brand + " Media Player: Media stopped");
    }
    
    public void setSubtitles(boolean enabled) {
        System.out.println(brand + " Media Player: Subtitles " + 
                          (enabled ? "enabled" : "disabled"));
    }
}

// Climate Control
class ClimateControl {
    private int temperature;
    
    public ClimateControl() {
        this.temperature = 72; // Default temperature
    }
    
    public void setTemperature(int temperature) {
        this.temperature = temperature;
        System.out.println("Climate Control: Temperature set to " + temperature + "°F");
    }
    
    public void setMovieMode() {
        setTemperature(68);
        System.out.println("Climate Control: Movie mode activated - optimal temperature set");
    }
    
    public void turnOff() {
        System.out.println("Climate Control: System turned off");
    }
}

// Facade Class - Home Theater Facade
class HomeTheaterFacade {
    private AudioSystem audioSystem;
    private VideoSystem videoSystem;
    private LightingSystem lightingSystem;
    private MediaPlayer mediaPlayer;
    private ClimateControl climateControl;
    
    public HomeTheaterFacade() {
        this.audioSystem = new AudioSystem("Bose");
        this.videoSystem = new VideoSystem("Sony");
        this.lightingSystem = new LightingSystem();
        this.mediaPlayer = new MediaPlayer("Apple TV");
        this.climateControl = new ClimateControl();
    }
    
    // Constructor with custom components
    public HomeTheaterFacade(AudioSystem audio, VideoSystem video, 
                           LightingSystem lighting, MediaPlayer player,
                           ClimateControl climate) {
        this.audioSystem = audio;
        this.videoSystem = video;
        this.lightingSystem = lighting;
        this.mediaPlayer = player;
        this.climateControl = climate;
    }
    
    // High-level operations
    public void watchMovie(String movieTitle) {
        System.out.println("\n=== Setting up for movie: " + movieTitle + " ===");
        
        // Power on all systems
        audioSystem.powerOn();
        videoSystem.powerOn();
        mediaPlayer.powerOn();
        
        // Configure audio
        audioSystem.setVolume(70);
        audioSystem.setSurroundSound(true);
        audioSystem.selectInput("HDMI1");
        
        // Configure video
        videoSystem.setResolution("4K");
        videoSystem.selectInput("HDMI1");
        videoSystem.adjustBrightness(30);
        
        // Set up lighting
        lightingSystem.setAmbientLighting();
        lightingSystem.dimLights(20);
        
        // Configure climate
        climateControl.setMovieMode();
        
        // Start media
        mediaPlayer.loadMedia("Movie", movieTitle);
        mediaPlayer.setSubtitles(true);
        mediaPlayer.play();
        
        System.out.println("=== Movie setup complete! Enjoy your movie! ===\n");
    }
    
    public void watchTV(String channel) {
        System.out.println("\n=== Setting up for TV: " + channel + " ===");
        
        audioSystem.powerOn();
        videoSystem.powerOn();
        
        audioSystem.setVolume(50);
        audioSystem.setSurroundSound(false);
        audioSystem.selectInput("Cable");
        
        videoSystem.setResolution("1080p");
        videoSystem.selectInput("Cable");
        videoSystem.adjustBrightness(50);
        
        lightingSystem.turnOnLights();
        
        System.out.println("=== TV setup complete! ===\n");
    }
    
    public void listenToMusic(String musicSource) {
        System.out.println("\n=== Setting up for music: " + musicSource + " ===");
        
        audioSystem.powerOn();
        mediaPlayer.powerOn();
        
        audioSystem.setVolume(60);
        audioSystem.setSurroundSound(true);
        audioSystem.selectInput("Bluetooth");
        
        lightingSystem.turnOnLights();
        
        mediaPlayer.loadMedia("Music", musicSource);
        mediaPlayer.play();
        
        System.out.println("=== Music setup complete! ===\n");
    }
    
    public void pauseMedia() {
        System.out.println("\n=== Pausing media ===");
        mediaPlayer.pause();
        System.out.println("=== Media paused ===\n");
    }
    
    public void resumeMedia() {
        System.out.println("\n=== Resuming media ===");
        mediaPlayer.play();
        System.out.println("=== Media resumed ===\n");
    }
    
    public void turnOffSystem() {
        System.out.println("\n=== Shutting down home theater system ===");
        
        mediaPlayer.stop();
        mediaPlayer.powerOff();
        audioSystem.powerOff();
        videoSystem.powerOff();
        lightingSystem.turnOffLights();
        climateControl.turnOff();
        
        System.out.println("=== System shutdown complete ===\n");
    }
    
    // Additional convenience methods
    public void setVolume(int volume) {
        audioSystem.setVolume(volume);
    }
    
    public void adjustLighting(int percentage) {
        lightingSystem.dimLights(percentage);
    }
    
    public void setTemperature(int temperature) {
        climateControl.setTemperature(temperature);
    }
}

// Client code demonstrating usage
public class FacadePatternDemo {
    public static void main(String[] args) {
        // Create the facade
        HomeTheaterFacade homeTheater = new HomeTheaterFacade();
        
        // Easy high-level operations
        homeTheater.watchMovie("The Matrix");
        
        // Pause and resume
        homeTheater.pauseMedia();
        homeTheater.resumeMedia();
        
        // Quick adjustments
        homeTheater.setVolume(80);
        homeTheater.adjustLighting(10);
        
        // Switch to TV
        homeTheater.watchTV("ESPN");
        
        // Listen to music
        homeTheater.listenToMusic("Spotify Playlist");
        
        // Shutdown
        homeTheater.turnOffSystem();
        
        System.out.println("\n" + "=".repeat(50));
        System.out.println("WITHOUT FACADE (Complex direct usage):");
        System.out.println("=".repeat(50));
        
        // Show how complex it would be without facade
        demonstrateWithoutFacade();
    }
    
    private static void demonstrateWithoutFacade() {
        // Without facade, client needs to know about all subsystems
        AudioSystem audio = new AudioSystem("Bose");
        VideoSystem video = new VideoSystem("Sony");
        LightingSystem lighting = new LightingSystem();
        MediaPlayer player = new MediaPlayer("Apple TV");
        ClimateControl climate = new ClimateControl();
        
        // Complex setup for watching a movie
        System.out.println("\nClient code without facade for watching a movie:");
        audio.powerOn();
        video.powerOn();
        player.powerOn();
        
        audio.setVolume(70);
        audio.setSurroundSound(true);
        audio.selectInput("HDMI1");
        
        video.setResolution("4K");
        video.selectInput("HDMI1");
        video.adjustBrightness(30);
        
        lighting.setAmbientLighting();
        lighting.dimLights(20);
        
        climate.setMovieMode();
        
        player.loadMedia("Movie", "The Matrix");
        player.setSubtitles(true);
        player.play();
        
        System.out.println("Movie setup complete (with lots of complex client code!)");
    }
}
```

### Java Output:
```
=== Setting up for movie: The Matrix ===
Bose Audio System: Powering on...
Sony Video System: Powering on...
Apple TV Media Player: Powering on...
Bose Audio System: Volume set to 70
Bose Audio System: Surround sound enabled
Bose Audio System: Input selected - HDMI1
Sony Video System: Resolution set to 4K
Sony Video System: Input selected - HDMI1
Sony Video System: Brightness adjusted to 30
Lighting System: Setting ambient lighting for movie watching
Lighting System: Dimming all lights to 20%
  - Living Room lights dimmed
  - Kitchen lights dimmed
  - Bedroom lights dimmed
Climate Control: Temperature set to 68°F
Climate Control: Movie mode activated - optimal temperature set
Apple TV Media Player: Loading Movie - The Matrix
Apple TV Media Player: Subtitles enabled
Apple TV Media Player: Playing media...
=== Movie setup complete! Enjoy your movie! ===

=== Pausing media ===
Apple TV Media Player: Media paused
=== Media paused ===

=== Resuming media ===
Apple TV Media Player: Playing media...
=== Media resumed ===

Bose Audio System: Volume set to 80
Lighting System: Dimming all lights to 10%
  - Living Room lights dimmed
  - Kitchen lights dimmed
  - Bedroom lights dimmed
```

## Implementation in Go

### Go Implementation - Home Theater System Example

```go
package main

import (
    "fmt"
    "strings"
)

// Subsystem Interfaces and Structs

// AudioSystem subsystem
type AudioSystem struct {
    Brand  string
    Volume int
}

func NewAudioSystem(brand string) *AudioSystem {
    return &AudioSystem{
        Brand:  brand,
        Volume: 0,
    }
}

func (a *AudioSystem) PowerOn() {
    fmt.Printf("%s Audio System: Powering on...\n", a.Brand)
}

func (a *AudioSystem) PowerOff() {
    fmt.Printf("%s Audio System: Powering off...\n", a.Brand)
}

func (a *AudioSystem) SetVolume(volume int) {
    a.Volume = volume
    fmt.Printf("%s Audio System: Volume set to %d\n", a.Brand, volume)
}

func (a *AudioSystem) SetSurroundSound(enabled bool) {
    status := "disabled"
    if enabled {
        status = "enabled"
    }
    fmt.Printf("%s Audio System: Surround sound %s\n", a.Brand, status)
}

func (a *AudioSystem) SelectInput(input string) {
    fmt.Printf("%s Audio System: Input selected - %s\n", a.Brand, input)
}

// VideoSystem subsystem
type VideoSystem struct {
    Brand      string
    Resolution string
}

func NewVideoSystem(brand string) *VideoSystem {
    return &VideoSystem{
        Brand:      brand,
        Resolution: "1080p",
    }
}

func (v *VideoSystem) PowerOn() {
    fmt.Printf("%s Video System: Powering on...\n", v.Brand)
}

func (v *VideoSystem) PowerOff() {
    fmt.Printf("%s Video System: Powering off...\n", v.Brand)
}

func (v *VideoSystem) SetResolution(resolution string) {
    v.Resolution = resolution
    fmt.Printf("%s Video System: Resolution set to %s\n", v.Brand, resolution)
}

func (v *VideoSystem) SelectInput(input string) {
    fmt.Printf("%s Video System: Input selected - %s\n", v.Brand, input)
}

func (v *VideoSystem) AdjustBrightness(brightness int) {
    fmt.Printf("%s Video System: Brightness adjusted to %d\n", v.Brand, brightness)
}

// LightingSystem subsystem
type LightingSystem struct {
    Zones []string
}

func NewLightingSystem() *LightingSystem {
    return &LightingSystem{
        Zones: []string{"Living Room", "Kitchen", "Bedroom"},
    }
}

func (l *LightingSystem) DimLights(percentage int) {
    fmt.Printf("Lighting System: Dimming all lights to %d%%\n", percentage)
    for _, zone := range l.Zones {
        fmt.Printf("  - %s lights dimmed\n", zone)
    }
}

func (l *LightingSystem) TurnOffLights() {
    fmt.Println("Lighting System: Turning off all lights")
    for _, zone := range l.Zones {
        fmt.Printf("  - %s lights off\n", zone)
    }
}

func (l *LightingSystem) TurnOnLights() {
    fmt.Println("Lighting System: Turning on all lights")
    for _, zone := range l.Zones {
        fmt.Printf("  - %s lights on\n", zone)
    }
}

func (l *LightingSystem) SetAmbientLighting() {
    fmt.Println("Lighting System: Setting ambient lighting for movie watching")
}

// MediaPlayer subsystem
type MediaPlayer struct {
    Brand     string
    IsPlaying bool
}

func NewMediaPlayer(brand string) *MediaPlayer {
    return &MediaPlayer{
        Brand:     brand,
        IsPlaying: false,
    }
}

func (m *MediaPlayer) PowerOn() {
    fmt.Printf("%s Media Player: Powering on...\n", m.Brand)
}

func (m *MediaPlayer) PowerOff() {
    fmt.Printf("%s Media Player: Powering off...\n", m.Brand)
}

func (m *MediaPlayer) LoadMedia(mediaType, title string) {
    fmt.Printf("%s Media Player: Loading %s - %s\n", m.Brand, mediaType, title)
}

func (m *MediaPlayer) Play() {
    m.IsPlaying = true
    fmt.Printf("%s Media Player: Playing media...\n", m.Brand)
}

func (m *MediaPlayer) Pause() {
    m.IsPlaying = false
    fmt.Printf("%s Media Player: Media paused\n", m.Brand)
}

func (m *MediaPlayer) Stop() {
    m.IsPlaying = false
    fmt.Printf("%s Media Player: Media stopped\n", m.Brand)
}

func (m *MediaPlayer) SetSubtitles(enabled bool) {
    status := "disabled"
    if enabled {
        status = "enabled"
    }
    fmt.Printf("%s Media Player: Subtitles %s\n", m.Brand, status)
}

// ClimateControl subsystem
type ClimateControl struct {
    Temperature int
}

func NewClimateControl() *ClimateControl {
    return &ClimateControl{
        Temperature: 72, // Default temperature
    }
}

func (c *ClimateControl) SetTemperature(temperature int) {
    c.Temperature = temperature
    fmt.Printf("Climate Control: Temperature set to %d°F\n", temperature)
}

func (c *ClimateControl) SetMovieMode() {
    c.SetTemperature(68)
    fmt.Println("Climate Control: Movie mode activated - optimal temperature set")
}

func (c *ClimateControl) TurnOff() {
    fmt.Println("Climate Control: System turned off")
}

// Facade Interface
type HomeTheaterFacadeInterface interface {
    WatchMovie(movieTitle string)
    WatchTV(channel string)
    ListenToMusic(musicSource string)
    PauseMedia()
    ResumeMedia()
    TurnOffSystem()
    SetVolume(volume int)
    AdjustLighting(percentage int)
    SetTemperature(temperature int)
}

// Facade Implementation
type HomeTheaterFacade struct {
    audioSystem    *AudioSystem
    videoSystem    *VideoSystem
    lightingSystem *LightingSystem
    mediaPlayer    *MediaPlayer
    climateControl *ClimateControl
}

// Constructor with default components
func NewHomeTheaterFacade() *HomeTheaterFacade {
    return &HomeTheaterFacade{
        audioSystem:    NewAudioSystem("Bose"),
        videoSystem:    NewVideoSystem("Sony"),
        lightingSystem: NewLightingSystem(),
        mediaPlayer:    NewMediaPlayer("Apple TV"),
        climateControl: NewClimateControl(),
    }
}

// Constructor with custom components
func NewHomeTheaterFacadeWithComponents(audio *AudioSystem, video *VideoSystem,
    lighting *LightingSystem, player *MediaPlayer, climate *ClimateControl) *HomeTheaterFacade {
    return &HomeTheaterFacade{
        audioSystem:    audio,
        videoSystem:    video,
        lightingSystem: lighting,
        mediaPlayer:    player,
        climateControl: climate,
    }
}

// High-level operations
func (h *HomeTheaterFacade) WatchMovie(movieTitle string) {
    fmt.Printf("\n=== Setting up for movie: %s ===\n", movieTitle)
    
    // Power on all systems
    h.audioSystem.PowerOn()
    h.videoSystem.PowerOn()
    h.mediaPlayer.PowerOn()
    
    // Configure audio
    h.audioSystem.SetVolume(70)
    h.audioSystem.SetSurroundSound(true)
    h.audioSystem.SelectInput("HDMI1")
    
    // Configure video
    h.videoSystem.SetResolution("4K")
    h.videoSystem.SelectInput("HDMI1")
    h.videoSystem.AdjustBrightness(30)
    
    // Set up lighting
    h.lightingSystem.SetAmbientLighting()
    h.lightingSystem.DimLights(20)
    
    // Configure climate
    h.climateControl.SetMovieMode()
    
    // Start media
    h.mediaPlayer.LoadMedia("Movie", movieTitle)
    h.mediaPlayer.SetSubtitles(true)
    h.mediaPlayer.Play()
    
    fmt.Println("=== Movie setup complete! Enjoy your movie! ===\n")
}

func (h *HomeTheaterFacade) WatchTV(channel string) {
    fmt.Printf("\n=== Setting up for TV: %s ===\n", channel)
    
    h.audioSystem.PowerOn()
    h.videoSystem.PowerOn()
    
    h.audioSystem.SetVolume(50)
    h.audioSystem.SetSurroundSound(false)
    h.audioSystem.SelectInput("Cable")
    
    h.videoSystem.SetResolution("1080p")
    h.videoSystem.SelectInput("Cable")
    h.videoSystem.AdjustBrightness(50)
    
    h.lightingSystem.TurnOnLights()
    
    fmt.Println("=== TV setup complete! ===\n")
}

func (h *HomeTheaterFacade) ListenToMusic(musicSource string) {
    fmt.Printf("\n=== Setting up for music: %s ===\n", musicSource)
    
    h.audioSystem.PowerOn()
    h.mediaPlayer.PowerOn()
    
    h.audioSystem.SetVolume(60)
    h.audioSystem.SetSurroundSound(true)
    h.audioSystem.SelectInput("Bluetooth")
    
    h.lightingSystem.TurnOnLights()
    
    h.mediaPlayer.LoadMedia("Music", musicSource)
    h.mediaPlayer.Play()
    
    fmt.Println("=== Music setup complete! ===\n")
}

func (h *HomeTheaterFacade) PauseMedia() {
    fmt.Println("\n=== Pausing media ===")
    h.mediaPlayer.Pause()
    fmt.Println("=== Media paused ===\n")
}

func (h *HomeTheaterFacade) ResumeMedia() {
    fmt.Println("\n=== Resuming media ===")
    h.mediaPlayer.Play()
    fmt.Println("=== Media resumed ===\n")
}

func (h *HomeTheaterFacade) TurnOffSystem() {
    fmt.Println("\n=== Shutting down home theater system ===")
    
    h.mediaPlayer.Stop()
    h.mediaPlayer.PowerOff()
    h.audioSystem.PowerOff()
    h.videoSystem.PowerOff()
    h.lightingSystem.TurnOffLights()
    h.climateControl.TurnOff()
    
    fmt.Println("=== System shutdown complete ===\n")
}

// Additional convenience methods
func (h *HomeTheaterFacade) SetVolume(volume int) {
    h.audioSystem.SetVolume(volume)
}

func (h *HomeTheaterFacade) AdjustLighting(percentage int) {
    h.lightingSystem.DimLights(percentage)
}

func (h *HomeTheaterFacade) SetTemperature(temperature int) {
    h.climateControl.SetTemperature(temperature)
}

// Functional approach for Go-idiomatic facade
type HomeTheaterOption func(*HomeTheaterFacade)

func WithAudioSystem(audio *AudioSystem) HomeTheaterOption {
    return func(h *HomeTheaterFacade) {
        h.audioSystem = audio
    }
}

func WithVideoSystem(video *VideoSystem) HomeTheaterOption {
    return func(h *HomeTheaterFacade) {
        h.videoSystem = video
    }
}

func WithLightingSystem(lighting *LightingSystem) HomeTheaterOption {
    return func(h *HomeTheaterFacade) {
        h.lightingSystem = lighting
    }
}

func WithMediaPlayer(player *MediaPlayer) HomeTheaterOption {
    return func(h *HomeTheaterFacade) {
        h.mediaPlayer = player
    }
}

func WithClimateControl(climate *ClimateControl) HomeTheaterOption {
    return func(h *HomeTheaterFacade) {
        h.climateControl = climate
    }
}

// Functional constructor
func NewHomeTheaterFacadeWithOptions(options ...HomeTheaterOption) *HomeTheaterFacade {
    facade := &HomeTheaterFacade{
        audioSystem:    NewAudioSystem("Default Audio"),
        videoSystem:    NewVideoSystem("Default Video"),
        lightingSystem: NewLightingSystem(),
        mediaPlayer:    NewMediaPlayer("Default Player"),
        climateControl: NewClimateControl(),
    }
    
    for _, option := range options {
        option(facade)
    }
    
    return facade
}

// Client code demonstrating usage
func main() {
    // Create the facade
    homeTheater := NewHomeTheaterFacade()
    
    // Easy high-level operations
    homeTheater.WatchMovie("The Matrix")
    
    // Pause and resume
    homeTheater.PauseMedia()
    homeTheater.ResumeMedia()
    
    // Quick adjustments
    homeTheater.SetVolume(80)
    homeTheater.AdjustLighting(10)
    
    // Switch to TV
    homeTheater.WatchTV("ESPN")
    
    // Listen to music
    homeTheater.ListenToMusic("Spotify Playlist")
    
    // Shutdown
    homeTheater.TurnOffSystem()
    
    fmt.Println(strings.Repeat("=", 50))
    fmt.Println("FUNCTIONAL OPTIONS APPROACH:")
    fmt.Println(strings.Repeat("=", 50))
    
    // Demonstrate functional options approach
    customHomeTheater := NewHomeTheaterFacadeWithOptions(
        WithAudioSystem(NewAudioSystem("Sonos")),
        WithVideoSystem(NewVideoSystem("Samsung")),
        WithMediaPlayer(NewMediaPlayer("Roku")),
    )
    
    customHomeTheater.WatchMovie("Inception")
    customHomeTheater.TurnOffSystem()
    
    fmt.Println(strings.Repeat("=", 50))
    fmt.Println("WITHOUT FACADE (Complex direct usage):")
    fmt.Println(strings.Repeat("=", 50))
    
    // Show how complex it would be without facade
    demonstrateWithoutFacade()
}

func demonstrateWithoutFacade() {
    // Without facade, client needs to know about all subsystems
    audio := NewAudioSystem("Bose")
    video := NewVideoSystem("Sony")
    lighting := NewLightingSystem()
    player := NewMediaPlayer("Apple TV")
    climate := NewClimateControl()
    
    // Complex setup for watching a movie
    fmt.Println("\nClient code without facade for watching a movie:")
    audio.PowerOn()
    video.PowerOn()
    player.PowerOn()
    
    audio.SetVolume(70)
    audio.SetSurroundSound(true)
    audio.SelectInput("HDMI1")
    
    video.SetResolution("4K")
    video.SelectInput("HDMI1")
    video.AdjustBrightness(30)
    
    lighting.SetAmbientLighting()
    lighting.DimLights(20)
    
    climate.SetMovieMode()
    
    player.LoadMedia("Movie", "The Matrix")
    player.SetSubtitles(true)
    player.Play()
    
    fmt.Println("Movie setup complete (with lots of complex client code!)")
}
```

### Go Output: