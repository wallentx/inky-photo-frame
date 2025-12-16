# 📖 Guide d'Installation Complet - Inky Photo Frame

## 📦 Matériel Requis

- **Raspberry Pi** (Zero 2W, 3B+, 4 ou 5) avec alimentation
- **Carte SD** (8GB minimum) avec Raspberry Pi OS installé
- **Écran Inky Impression 7.3"** (800x480 pixels)
- **Connexion WiFi** configurée sur le Pi

## 🚀 Installation Rapide (5 minutes)

### Méthode 1 : Installation automatique depuis GitHub

```bash
# Connectez-vous en SSH à votre Raspberry Pi
ssh <utilisateur>@[ip-de-votre-pi]

# Téléchargez et lancez l'installateur
curl -sSL https://raw.githubusercontent.com/mehdi7129/inky-photo-frame/main/install.sh | bash
```

### Méthode 2 : Installation manuelle

```bash
# 1. Clonez le dépôt
git clone https://github.com/mehdi7129/inky-photo-frame.git
cd inky-photo-frame

# 2. Lancez l'installation
chmod +x install.sh
./install.sh
```

L'installation va :
- ✅ Installer toutes les dépendances
- ✅ Démarrer automatiquement au boot

## 📸 Synchronisation des photos

Après l'installation, synchronisez/copiez vos photos dans :

- `$HOME/Images`

Les nouvelles photos sont détectées automatiquement et s'affichent sans redémarrer le service.

Au premier démarrage, l'écran affiche l'adresse IP et le dossier des photos.

## 🎨 Fonctionnement

### Rotation des photos
- **Nouvelle photo :** S'affiche immédiatement quand ajoutée
- **Rotation quotidienne :** Change automatiquement à 5h du matin
- **Historique intelligent :** Ne répète jamais les photos jusqu'à avoir tout montré

### Formats supportés
- ✅ JPEG/JPG
- ✅ PNG
- ✅ HEIC (photos iPhone)
- ✅ GIF
- ✅ BMP

### Optimisation automatique
- Recadrage intelligent pour l'écran 800x480
- Ajustement du contraste pour e-ink
- Traitement des images portrait/paysage

## 🎮 Contrôles Physiques (Boutons)

L'Inky Impression dispose de **4 boutons physiques** sur le côté pour un contrôle interactif :

| Bouton | Position | Fonction |
|--------|----------|----------|
| **A** | Haut | ⏭️ Photo suivante |
| **B** | | ⏮️ Photo précédente |
| **C** | | 🎨 Cycle modes couleur |
| **D** | Bas | 🔄 Reset mode pimoroni |

### Modes de couleur disponibles
1. **pimoroni** (par défaut) - Rendu standard Pimoroni
2. **spectra_palette** - Palette calibrée 6 couleurs pour Spectra
3. **warmth_boost** - Boost chaleur agressif pour tons chauds

### Caractéristiques
- ✅ Aucun message affiché - actions silencieuses
- ✅ Boutons verrouillés pendant l'affichage (~30-40s)
- ✅ Préférence de couleur sauvegardée et persistante
- ✅ Navigation sans interface externe

## 🛠 Commandes Utiles

```bash
# Voir le statut
sudo systemctl status inky-photo-frame

# Voir les logs en temps réel
sudo journalctl -u inky-photo-frame -f

# Redémarrer le service
sudo systemctl restart inky-photo-frame

# Voir l'historique des photos
cat ~/.inky_history.json | python3 -m json.tool
```

## ❓ Résolution de Problèmes

### L'écran ne s'allume pas
```bash
# Vérifiez que le service tourne
sudo systemctl status inky-photo-frame

# Vérifiez les connexions de l'écran
# Pin 1 (3.3V), Pin 6 (GND), pins SPI activés
```

### Les photos ne s'affichent pas
1. Vérifiez le format (JPG, PNG, HEIC)
2. Vérifiez les logs : `tail -f "$HOME/inky_photo_frame.log"`
3. Vérifiez les permissions : `ls -la "$HOME/Images"`

## 🗑 Désinstallation

Pour retirer complètement l'application :
```bash
cd inky-photo-frame
./uninstall.sh
```

## 📝 Configuration Avancée

Éditez `$HOME/inky-photo-frame/inky_photo_frame.py` :

```python
CHANGE_HOUR = 5  # Heure de changement quotidien (0-23)
PHOTOS_DIR = Path.home() / "Images"  # Dossier des photos
```

## 🆘 Support

- **GitHub Issues :** [github.com/mehdi7129/inky-photo-frame/issues](https://github.com/mehdi7129/inky-photo-frame/issues)
- **Documentation :** [github.com/mehdi7129/inky-photo-frame](https://github.com/mehdi7129/inky-photo-frame)

## 💡 Astuces

1. **Organisation des photos :** Créez des sous-dossiers par événement
2. **Qualité optimale :** Utilisez des photos de 800x480 pixels minimum
3. **Économie d'énergie :** L'écran e-ink ne consomme que lors du changement
4. **Sauvegarde :** L'historique est dans `~/.inky_history.json`

---

**Profitez de votre cadre photo numérique !** 📷✨
