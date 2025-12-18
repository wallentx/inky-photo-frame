# GitHub Copilot Instructions for Inky Photo Frame

## Project Overview

This is a digital photo frame application for Raspberry Pi with Inky Impression e-ink displays. It displays photos from a local photos directory with smart rotation and supports multiple display models through automatic detection.

**Key Features:**
- Auto-detects Inky display models (7.3" and 13.3", various editions)
- Displays photos from a local directory with immediate updates
- Daily photo rotation at 5AM with intelligent history tracking
- Physical button controls for navigation and color mode switching
- Multiple color modes optimized for different e-ink displays
- Ultra-low power consumption (~0.6W average)
- Supports HEIC image format from modern phones

## Technology Stack

- **Language:** Python 3
- **Hardware:** Raspberry Pi (Zero 2W, 3, 4, or 5) with Inky Impression displays
- **Key Libraries:**
  - `inky` - Pimoroni's official library for e-ink display control
  - `Pillow` (PIL) - Image processing and manipulation
  - `pillow-heif` - iPhone HEIC format support
  - `watchdog` - File system monitoring for instant photo updates
  - `gpiozero`, `RPi.GPIO`, `lgpio` - Physical button support
- **Services:** systemd service (`inky-photo-frame.service`) for auto-start
- **Sync:** External sync tools can write into the photos directory (e.g. Google Photos cron job, rsync, scp)

## Project Structure

```
inky-photo-frame/
├── inky_photo_frame.py      # Main application (1000+ lines)
├── inky-photo-frame-cli     # CLI wrapper script
├── install.sh               # Automated installer
├── uninstall.sh            # Automated uninstaller
├── update.sh               # Update script
├── diagnostic_report.sh    # Diagnostic and troubleshooting tool
├── pyproject.toml          # Python dependencies and project metadata
├── logrotate.conf         # Log rotation configuration
├── README.md              # User documentation
├── INSTALLATION_GUIDE.md  # Detailed installation guide
├── COLOR_CALIBRATION.md   # Color mode documentation
├── CHANGELOG.md           # Version history
└── .github/
    └── copilot-instructions.md  # This file
```

## Architecture & Code Organization

### Main Application (`inky_photo_frame.py`)

The application is organized into these key sections:

1. **Configuration Constants** (lines 58-96)
   - Paths, timing settings, color modes
   - Default: `COLOR_MODE = 'spectra_palette'`

2. **Color Management Functions**
   - `apply_color_mode()` - Applies selected color processing to images
   - `quantize_to_spectra_palette()` - Maps images to 6-color e-ink palette
   - `apply_warmth_boost()` - Warm color enhancement for portraits
   - `apply_pimoroni_mode()` - Official Pimoroni default processing

3. **Core Display Logic**
   - `PhotoFrame` class - Main application controller
   - `display_photo()` - Renders images to e-ink display
   - `display_welcome_screen()` - Shows network setup info on first boot

4. **Photo Management**
   - `get_random_photo()` - Selects next photo with history tracking
   - `schedule_next_change()` - Manages daily 5AM rotation
   - File monitoring via `watchdog` for instant new photo display

5. **Hardware Integration**
   - Physical button handlers (A/B/C/D buttons on Inky)
   - GPIO initialization with fallback support
   - E-ink refresh control (30-40 second refresh cycle)

### Installation Scripts

- **install.sh**: Enables I2C/SPI, installs dependencies, creates systemd service, disables LEDs
- **uninstall.sh**: Removes service and installed files
- **update.sh**: Git-based updater with service restart

## Development Guidelines

### Code Style

- Follow PEP 8 conventions
- Use descriptive variable names (e.g., `CHANGE_HOUR`, `PHOTOS_DIR`)
- Add docstrings to functions explaining purpose and behavior
- Keep functions focused and modular
- Use Path objects from pathlib for file paths

### Hardware Considerations

**IMPORTANT - Always keep in mind:**

1. **E-ink Display Constraints:**
   - Refresh takes 30-40 seconds and blocks during this time
   - Limited color palette (6-7 colors depending on model)
   - Display retains image when powered off (persistent display)
   - Avoid rapid refreshes (can damage display)

2. **GPIO/Button Handling:**
   - Buttons are physical hardware on Inky Impression
   - Must handle button presses during display refresh (lock during refresh)
   - Support multiple GPIO backends (lgpio, RPi.GPIO)
   - Always include try/except for GPIO import failures

3. **Raspberry Pi Specifics:**
   - I2C and SPI must be enabled for display to work
   - Runs as a systemd service under the install user
   - Uses the current user's home directory (`$HOME`, typically `/home/$USER`)
   - Consider memory constraints on Pi Zero models

### Image Processing

- **Auto-detect display** using `inky.auto()` - don't hardcode display models
- **Preserve aspect ratios** when possible or use smart cropping
- **Color modes** are user-configurable - respect the `COLOR_MODE` setting
- **Support multiple formats**: JPEG, PNG, HEIC, WEBP
- **Optimize for e-ink**: Reduce gradients, increase contrast
- **Test with various image sizes**: phones produce high-resolution images

### Testing & Validation

Since this is a hardware-dependent project running on Raspberry Pi:

1. **Manual Testing Required:**
   - Test on actual Raspberry Pi with Inky display
   - Verify button functionality (A, B, C, D)
   - Test different image formats (JPEG, PNG, HEIC)
   - Validate color modes on actual e-ink display

2. **Service Management:**
   ```bash
   # Check service status
   sudo systemctl status inky-photo-frame
   
   # View logs
   sudo journalctl -u inky-photo-frame -f
   
   # Restart after changes
   sudo systemctl restart inky-photo-frame
   ```

3. **Common Test Scenarios:**
   - Add new photo → verify immediate display
   - Wait for 5AM rotation → verify scheduled change
   - Press buttons → verify navigation and color mode switching
   - Check photo history → ensure no duplicate displays
   - Monitor memory usage → especially on Pi Zero

### Error Handling

- **Graceful degradation**: If GPIO fails, continue without buttons
- **File handling**: Use try/except for image loading (corrupt files)
- **Network resilience**: Handle missing photos directory gracefully
- **Logging**: Use logging module, not print statements
- **Display errors on screen**: Welcome screen shows helpful diagnostics

### Configuration Files

The application uses JSON files in `$HOME/`:

- `.inky_history.json` - Tracks displayed photos (prevents repeats)
- `.inky_color_mode.json` - Stores user's color mode preference
- `inky_photo_frame.log` - Application logs (with rotation)

### Security Considerations

- **File permissions**: Ensure proper ownership (service user)

## Common Modification Patterns

### Adding a New Color Mode

1. Add configuration constants at top of file
2. Create `apply_[mode_name]()` function
3. Update `apply_color_mode()` to handle new mode
4. Update CLI/button handlers to support new mode
5. Document in COLOR_CALIBRATION.md

### Changing Display Behavior

- **Photo rotation time**: Modify `CHANGE_HOUR` constant
- **Photo directory**: Update `PHOTOS_DIR` path
- **History size**: Adjust history tracking logic
- **Button actions**: Modify button callback functions

### Supporting New Hardware

- **New display models**: Should auto-detect via `inky.auto()`
- **Different GPIO layouts**: Update button pin numbers
- **Alternative platforms**: Consider path and service differences

## File Paths & Conventions

- **Always use absolute paths** - code runs as systemd service
- **Installation directory**: `$HOME/inky-photo-frame/`
- **Photos directory**: `$HOME/Images/`
- **Configuration files**: `$HOME/.inky_*`
- **Logs**: `$HOME/inky_photo_frame.log`

## Dependencies & Version Management

- Pin major versions in `pyproject.toml` (e.g., `>=1.5.0`)
- Test compatibility with Raspberry Pi OS Bookworm (current)
- Support both legacy and modern GPIO libraries
- Document any system-level dependencies (I2C, SPI)

## Documentation Standards

- **README.md**: User-facing, installation and usage
- **INSTALLATION_GUIDE.md**: Detailed setup steps
- **COLOR_CALIBRATION.md**: Technical color mode details
- **CHANGELOG.md**: Version history with semantic versioning
- **Code comments**: Explain "why", not "what"

## Best Practices for This Project

1. **Minimize dependencies**: Every library must be justified
2. **Respect e-ink limitations**: Slow refresh, limited colors
3. **Optimize for low power**: This runs 24/7 on battery-capable devices
4. **User-friendly errors**: Display helpful messages on screen
5. **Preserve user settings**: Color modes, history persist across reboots
6. **Support all Inky models**: Use auto-detection, not hardcoding
7. **Test on real hardware**: Emulation cannot replicate e-ink behavior

## Troubleshooting Knowledge

Common issues developers should be aware of:

- **"No Inky display detected"** → Check I2C/SPI enabled, physical connection
- **Buttons not working** → GPIO library conflicts, check imports
- **Photos not appearing** → Check the photos directory permissions and file format support
- **Slow performance** → Image too large, optimize before processing
- **Color looks wrong** → Try different COLOR_MODE settings
- **Service won't start** → Check logs, Python dependencies, file permissions

## Contributing Guidelines

When making changes:

1. **Test on actual hardware** before submitting
2. **Update documentation** if behavior changes
3. **Maintain backward compatibility** with user configurations
4. **Follow existing code patterns** for consistency
5. **Document hardware requirements** for new features
6. **Consider Pi Zero performance** (slowest supported device)

## File location notice

This file is deprecated and kept only for backward compatibility with older tooling.

The canonical GitHub Copilot instructions for this project are located at:
`.github/copilot-instructions.md`.

Please update any references or documentation to point to the new path.
