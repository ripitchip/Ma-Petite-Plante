# 🌱 Micro-Wiki : Ma Petite Plante

## Development

### Setup

```bash
# Créer l'émulateur
echo "no" | avdmanager create avd -n nothing_2a -k "system-images;android-34;google_apis;x86_64" --force

```

### Run

1. **Hôte (NixOS) :** `xhost +local:docker`
2. **Émulateur (VS Code) :**
```bash
emulator -avd nothing_2a -no-audio -gpu swiftshader_indirect -skin 542x1206 -scale 1.0

```