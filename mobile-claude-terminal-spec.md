# Mobile Terminal System for Claude Code Remote Access
## Complete Project Specification

---

## Instructions for Claude Code

**Before beginning implementation, please:**

1. **Read the MCP Builder skill** to understand how to structure this project as a well-designed system with clear interfaces between components
   - Location: `/mnt/skills/examples/mcp-builder/SKILL.md`
   - Apply the architectural principles even though this isn't an MCP server

2. **Reference the Skill Creator skill** to understand best practices for creating reusable, well-documented code
   - Location: `/mnt/skills/examples/skill-creator/SKILL.md`
   - Focus on the sections about clear documentation and examples

3. **Create a phased implementation plan** before writing any code:
   - Phase 1: Go TUI application with basic setup wizard
   - Phase 2: Core functionality (SSH, Mosh, Tailscale integration)
   - Phase 3: Dashboard and monitoring
   - Phase 4: iOS app foundation
   - Phase 5: iOS networking and connection management
   - Phase 6: Polish and testing

4. **Follow these development principles:**
   - Write production-quality code with proper error handling
   - Include comprehensive comments and documentation
   - Create example configurations
   - Build with testing in mind
   - Make it easy to extend and maintain

---

## Project Overview

Create a secure, beautiful, and resilient terminal system that allows remote access to Claude Code on a Mac from an iPhone while away from the desk. The system uses Tailscale for secure peer-to-peer networking and Mosh for resilient mobile connections that handle network changes gracefully.

### Core Value Proposition
- **For the user**: Work with Claude Code from anywhere using just an iPhone
- **Security first**: All traffic through Tailscale's encrypted mesh, no exposed ports
- **Resilience**: Mosh keeps connections alive through network changes
- **Beautiful UX**: Polished TUI on Mac, native SwiftUI on iOS

---

## Component 1: macOS TUI Application (Go + Bubble Tea)

### Technology Stack
- **Framework**: Bubble Tea (github.com/charmbracelet/bubbletea)
- **Styling**: Lip Gloss (github.com/charmbracelet/lipgloss)
- **Components**: Bubbles (github.com/charmbracelet/bubbles)
- **CLI Framework**: Cobra (optional, for command structure)
- **Language**: Go 1.21+

### Core Features

#### 1. Setup Wizard (Primary Feature)
An interactive, step-by-step TUI wizard that guides users through initial configuration:

**Screens:**
1. **Welcome Screen**
   - Project description
   - What will be installed/configured
   - Estimated time (5-10 minutes)
   - Continue/Exit options

2. **System Check Screen**
   - Detect if Homebrew is installed
   - Check for existing Tailscale installation
   - Check for existing Mosh installation
   - Check SSH server status
   - Display results with colored indicators (✓/✗)

3. **Dependency Installation Screen**
   - Interactive prompts for each missing dependency
   - Real-time output from Homebrew installs
   - Progress indicators
   - Error handling with retry options

4. **Tailscale Configuration Screen**
   - Check if Tailscale is authenticated
   - Display current Tailscale status
   - Show machine name and IP
   - Prompt to run `tailscale up` if needed

5. **SSH Configuration Screen**
   - Enable SSH server if disabled
   - Check for existing SSH keys
   - Generate new keys if needed (with user consent)
   - Display public key for reference
   - Configure SSH daemon for optimal settings

6. **Mosh Configuration Screen**
   - Verify Mosh installation
   - Configure Mosh server settings
   - Set up firewall rules (UDP 60000-61000)
   - Test Mosh server launch

7. **Firewall Configuration Screen**
   - Display required firewall rules
   - Apply rules for Mosh ports
   - Verify rules are active
   - Provide manual instructions if automated setup fails

8. **Summary Screen**
   - Display all configuration details
   - Show connection information
   - Tailscale hostname
   - Username
   - SSH key fingerprint
   - Option to generate QR code
   - Option to export config file

#### 2. Dashboard View (Monitoring Interface)
A real-time status dashboard with multiple panels:

**Layout:**
```
┌─────────────────────────────────────────────────────────────┐
│ Mobile Claude Terminal - Dashboard              [Q]uit [R]efresh │
├─────────────────────────────────────────────────────────────┤
│ System Status                                                │
│ ┌─────────────────┬─────────────────┬──────────────────┐   │
│ │ Tailscale       │ SSH Server      │ Mosh Server       │   │
│ │ ✓ Connected     │ ✓ Running       │ ✓ Ready          │   │
│ │ 100.64.0.2      │ Port 22         │ Ports 60000-61000│   │
│ └─────────────────┴─────────────────┴──────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│ Active Sessions                                              │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ iPhone (iOS) - 100.64.0.5                             │   │
│ │ Connected: 2h 34m  •  Packets: 15.2k  •  Mosh PID: 1234│   │
│ └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│ Recent Activity                                              │
│ • 14:23 - New connection from iPhone                         │
│ • 14:22 - Mosh session started (port 60001)                 │
│ • 13:45 - Previous session ended                             │
└─────────────────────────────────────────────────────────────┘
```

**Features:**
- Real-time updates (refresh every 2-5 seconds)
- Color-coded status indicators
- Session details (duration, data transferred, client info)
- Activity log with timestamps
- Keyboard shortcuts for common actions
- Responsive layout

#### 3. QR Code Generator
Generate QR codes containing connection information for easy iOS setup:

**QR Code Content (JSON):**
```json
{
  "version": "1.0",
  "hostname": "macbook-pro.tail-scale.ts.net",
  "username": "john",
  "ssh_key_fingerprint": "SHA256:abc123...",
  "mosh_ports": "60000-61000",
  "created_at": "2025-10-17T14:23:00Z"
}
```

**Display:**
- Render QR code in terminal using Unicode box drawing characters
- Display alongside human-readable connection details
- Option to save QR code as PNG file
- Include instructions for iOS app

#### 4. Configuration Export
Generate configuration files for the iOS app:

**Config File Format (JSON):**
```json
{
  "version": "1.0",
  "profiles": [
    {
      "name": "My MacBook Pro",
      "hostname": "macbook-pro.tail-scale.ts.net",
      "tailscale_ip": "100.64.0.2",
      "username": "john",
      "ssh_port": 22,
      "mosh_ports": "60000-61000",
      "ssh_public_key": "ssh-ed25519 AAAA...",
      "ssh_key_fingerprint": "SHA256:abc123...",
      "created_at": "2025-10-17T14:23:00Z"
    }
  ]
}
```

**Export Options:**
- Save to ~/Downloads/
- Save to custom location
- Display file path
- Instructions for transferring to iOS (AirDrop, iCloud, etc.)

#### 5. Session Management
View and manage active Mosh sessions:

**Features:**
- List all active sessions with details
- Show connection duration, client IP, PID
- Kill individual sessions
- View session logs
- Filter/search sessions

#### 6. Configuration Management
Edit and manage system configuration:

**Settings:**
- Mosh port range
- SSH settings
- Tailscale preferences
- Log levels
- Auto-start options
- Security settings

#### 7. Logs Viewer
View detailed logs for troubleshooting:

**Log Types:**
- Connection logs
- SSH logs
- Mosh server logs
- System logs
- Error logs

**Features:**
- Real-time log streaming
- Filter by log level
- Search functionality
- Export logs to file

### Command Line Interface

```bash
mobile-claude-terminal [command]

Commands:
  setup           Run the interactive setup wizard
  dashboard       Show the live status dashboard (default)
  generate-qr     Generate QR code for iOS app setup
  export-config   Export configuration file for iOS app
  sessions        Manage active Mosh sessions
  config          Configure application settings
  logs            View application logs
  status          Show quick status summary
  version         Show version information
  help            Show help information

Flags:
  -h, --help      Help for any command
  --config-dir    Specify custom config directory
  --verbose       Enable verbose output
```

### Technical Implementation Details

#### Project Structure
```
mobile-claude-terminal/
├── cmd/
│   └── main.go                 # Entry point
├── internal/
│   ├── tui/
│   │   ├── setup/             # Setup wizard screens
│   │   ├── dashboard/         # Dashboard view
│   │   ├── sessions/          # Session management
│   │   ├── config/            # Configuration UI
│   │   └── common/            # Shared TUI components
│   ├── system/
│   │   ├── tailscale.go       # Tailscale operations
│   │   ├── ssh.go             # SSH configuration
│   │   ├── mosh.go            # Mosh management
│   │   ├── homebrew.go        # Homebrew operations
│   │   └── firewall.go        # Firewall rules
│   ├── config/
│   │   ├── config.go          # Configuration management
│   │   └── export.go          # Config export
│   └── qr/
│       └── generator.go       # QR code generation
├── pkg/
│   └── models/                # Shared data models
├── go.mod
├── go.sum
├── README.md
└── LICENSE
```

#### Key Dependencies
```go
require (
    github.com/charmbracelet/bubbletea v0.25.0
    github.com/charmbracelet/lipgloss v0.9.1
    github.com/charmbracelet/bubbles v0.17.1
    github.com/spf13/cobra v1.8.0
    github.com/skip2/go-qrcode v0.0.0-20200617195104-da1b6568686e
    // Add other dependencies as needed
)
```

#### Error Handling
- Graceful degradation when services unavailable
- Clear error messages with suggested solutions
- Retry mechanisms for network operations
- Logging of all errors for debugging

#### Configuration Storage
- Location: `~/.config/mobile-claude-terminal/`
- Files:
  - `config.json` - Main configuration
  - `sessions.json` - Active session data
  - `logs/` - Log files

---

## Component 2: iOS Terminal App (Swift/SwiftUI)

### Technology Stack
- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI
- **Minimum iOS**: iOS 16.0+
- **SSH Library**: NMSSH or Citadel
- **Mosh**: Custom wrapper or libmosh integration
- **Keychain**: For secure credential storage

### Core Features

#### 1. Onboarding & Setup

**Welcome Screen:**
- App introduction
- Feature highlights
- Privacy information
- Setup options (QR code vs manual)

**QR Code Scanner:**
- Camera permission request
- Real-time QR code detection
- Parse connection JSON
- Validate configuration
- Save profile automatically

**Manual Setup:**
- Form-based input
- Field validation
- Tailscale hostname discovery
- SSH key import options
- Test connection button

**Config File Import:**
- Files app integration
- AirDrop support
- Parse and validate JSON
- Create connection profile

#### 2. Terminal Interface

**Main Terminal View:**
```
┌─────────────────────────────────────┐
│ MacBook Pro  [Connected] ●          │
├─────────────────────────────────────┤
│                                      │
│ $ claude-code                        │
│ Welcome to Claude Code!              │
│                                      │
│ What would you like to build?        │
│ >_                                   │
│                                      │
│                                      │
│                                      │
│                                      │
├─────────────────────────────────────┤
│ [^] [Esc] [Tab] [⌘] [←] [→] [Clear] │
└─────────────────────────────────────┘
```

**Terminal Features:**
- VT100/xterm terminal emulation
- ANSI color support (256 colors)
- Unicode character support
- Smooth scrolling
- Text selection and copy
- Long-press for paste
- Pinch to zoom font size
- Swipe gestures for common actions

**Keyboard Toolbar:**
- Customizable quick keys
- Sticky modifier keys (Ctrl, Alt, Cmd)
- Common shortcuts (Esc, Tab, Arrow keys)
- Quick clear screen button
- Dismiss keyboard button

#### 3. Connection Management

**Profile Management:**
- Multiple connection profiles
- Edit/delete profiles
- Favorite profiles
- Recently used
- Connection status indicators

**Connection Flow:**
1. Select profile
2. Establish Tailscale connection
3. Initiate SSH connection
4. Start Mosh session
5. Present terminal interface

**Auto-Reconnection:**
- Detect connection loss
- Automatic retry with backoff
- Network quality monitoring
- Connection status notifications

#### 4. Settings & Customization

**Appearance:**
- Terminal themes (Dracula, Solarized, etc.)
- Font selection (SF Mono, JetBrains Mono, Fira Code)
- Font size adjustment
- Color scheme customization
- Dark/Light mode support

**Behavior:**
- Haptic feedback toggle
- Sound effects toggle
- Auto-reconnect settings
- Background connection timeout
- Keyboard settings

**Advanced:**
- SSH settings (compression, keepalive)
- Mosh settings (prediction mode)
- Port ranges
- Debug logging

#### 5. Security Features

**Authentication:**
- SSH key-based authentication
- Secure key storage in Keychain
- Key passphrase support
- Biometric authentication for app access (optional)

**Privacy:**
- No logging of sensitive data
- Secure credential storage
- Session encryption
- Clear explanation of permissions

### UI/UX Design Principles

1. **Native iOS Feel**: Use standard iOS patterns and gestures
2. **Accessibility**: Support VoiceOver, Dynamic Type, reduce motion
3. **Performance**: 60fps scrolling, efficient rendering
4. **Battery Efficiency**: Optimize network usage, background processing
5. **Error Recovery**: Clear error messages, recovery suggestions

### Technical Implementation Details

#### Project Structure
```
MobileClaudeTerminal/
├── MobileClaudeTerminal/
│   ├── App/
│   │   ├── MobileClaudeTerminalApp.swift
│   │   └── AppDelegate.swift
│   ├── Views/
│   │   ├── Onboarding/
│   │   │   ├── WelcomeView.swift
│   │   │   ├── QRScannerView.swift
│   │   │   └── ManualSetupView.swift
│   │   ├── Terminal/
│   │   │   ├── TerminalView.swift
│   │   │   ├── KeyboardToolbar.swift
│   │   │   └── TerminalTextView.swift
│   │   ├── Profiles/
│   │   │   ├── ProfileListView.swift
│   │   │   └── ProfileDetailView.swift
│   │   └── Settings/
│   │       ├── SettingsView.swift
│   │       ├── AppearanceSettingsView.swift
│   │       └── AdvancedSettingsView.swift
│   ├── Models/
│   │   ├── ConnectionProfile.swift
│   │   ├── TerminalSession.swift
│   │   └── AppSettings.swift
│   ├── Services/
│   │   ├── SSHService.swift
│   │   ├── MoshService.swift
│   │   ├── TailscaleService.swift
│   │   └── KeychainService.swift
│   ├── Utilities/
│   │   ├── TerminalEmulator.swift
│   │   ├── QRCodeParser.swift
│   │   └── ConfigImporter.swift
│   └── Resources/
│       ├── Assets.xcassets
│       ├── Fonts/
│       └── Themes/
├── MobileClaudeTerminalTests/
└── MobileClaudeTerminal.xcodeproj
```

#### Key Dependencies (SPM)
```swift
dependencies: [
    .package(url: "https://github.com/Frugghi/SwiftSSH.git", from: "1.0.0"),
    // Or alternative SSH library
    // Add Mosh wrapper/library
    // Add QR code scanning library
]
```

#### Data Models

**ConnectionProfile:**
```swift
struct ConnectionProfile: Codable, Identifiable {
    let id: UUID
    var name: String
    var hostname: String
    var tailscaleIP: String?
    var username: String
    var sshPort: Int
    var moshPorts: String
    var sshKeyFingerprint: String
    var createdAt: Date
    var lastUsed: Date?
    var isFavorite: Bool
}
```

**TerminalSession:**
```swift
class TerminalSession: ObservableObject {
    @Published var isConnected: Bool
    @Published var connectionQuality: ConnectionQuality
    @Published var buffer: TerminalBuffer
    let profile: ConnectionProfile
    // SSH/Mosh connection objects
    // Terminal emulator
}
```

---

## Networking & Security Architecture

### Tailscale Integration

**macOS (Go app):**
- Execute `tailscale status` to get machine info
- Parse JSON output for IP addresses
- Monitor Tailscale daemon status
- Provide status in dashboard

**iOS (Swift app):**
- Option 1: Use Tailscale SDK if available
- Option 2: Detect Tailscale VPN connection
- Option 3: Manual IP entry with validation
- Use Tailscale hostnames for connection

### SSH Configuration

**Key Generation:**
- Generate Ed25519 keys (modern, secure)
- Store private key securely
- Display public key for iOS transfer

**Server Configuration:**
- Enable SSH if disabled
- Configure sshd for optimal performance
- Set up authorized_keys
- Configure keepalive settings

**iOS Client:**
- Import private key securely
- Store in Keychain with encryption
- Support for passphrase-protected keys
- Automatic key selection per profile

### Mosh Protocol

**Why Mosh:**
- Handles IP changes seamlessly
- Works on poor connections
- Low latency user interface
- Resumes after network interruption

**Configuration:**
- UDP ports 60000-61000
- Predictive echo for responsiveness
- Automatic server selection
- Fallback to SSH if Mosh unavailable

**Firewall Rules:**
```bash
# Allow Mosh UDP ports
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/local/bin/mosh-server
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblock /usr/local/bin/mosh-server
```

---

## Testing Requirements

### macOS TUI App Testing

**Unit Tests:**
- Configuration parsing
- System command execution
- Status detection
- QR code generation
- Config export

**Integration Tests:**
- Homebrew installation simulation
- SSH configuration changes
- Firewall rule application
- Tailscale status parsing

**Manual Tests:**
- [ ] Clean install on new Mac
- [ ] Setup wizard completion
- [ ] Dashboard displays correctly
- [ ] QR code generation works
- [ ] Config export is valid
- [ ] All commands functional
- [ ] Error handling graceful

### iOS App Testing

**Unit Tests:**
- QR code parsing
- Config file import
- Connection profile management
- Keychain operations
- Terminal emulator

**Integration Tests:**
- SSH connection flow
- Mosh session management
- Network reconnection
- Background/foreground transitions

**UI Tests:**
- Onboarding flow
- Terminal interaction
- Settings changes
- Profile management

**Manual Tests:**
- [ ] QR code scanning works
- [ ] Manual setup completes
- [ ] Connection established
- [ ] Terminal rendering accurate
- [ ] Keyboard input correct
- [ ] Copy/paste works
- [ ] Network changes handled
- [ ] App backgrounding works
- [ ] Battery usage reasonable
- [ ] Claude Code works perfectly

---

## Documentation Requirements

### 1. README.md (Root)
```markdown
# Mobile Claude Terminal

Access Claude Code from your iPhone with a beautiful, secure terminal.

## Features
- Secure connections via Tailscale
- Resilient sessions with Mosh
- Beautiful TUI for macOS setup
- Native iOS terminal app
- One-tap QR code setup

## Quick Start
1. macOS: Run setup wizard
2. iOS: Scan QR code
3. Connect and code!

[Detailed documentation links]
```

### 2. macOS App Documentation
- Installation guide
- Setup wizard walkthrough
- Dashboard usage
- Troubleshooting guide
- Advanced configuration
- Development setup

### 3. iOS App Documentation
- Installation guide (TestFlight/App Store)
- Setup via QR code
- Manual setup steps
- Terminal usage guide
- Tips for Claude Code
- Troubleshooting
- Privacy policy

### 4. Technical Documentation
- Architecture overview
- Component interaction diagram
- Security model
- Protocol details
- API documentation
- Contributing guide

### 5. User Guides
- First-time setup (step-by-step with screenshots)
- Common tasks
- Tips and tricks
- FAQ
- Known issues

---

## Development Workflow

### Phase 1: macOS TUI Foundation (Week 1)
1. Set up Go project structure
2. Implement basic TUI with Bubble Tea
3. Create welcome screen
4. Build system detection functionality
5. Test on clean macOS installation

**Deliverable:** Basic TUI that can detect system state

### Phase 2: Setup Wizard (Week 2)
1. Implement dependency installation
2. Add Tailscale configuration
3. Add SSH configuration
4. Add Mosh configuration
5. Add firewall configuration
6. Create summary screen

**Deliverable:** Complete setup wizard that configures system

### Phase 3: Dashboard & Monitoring (Week 3)
1. Build dashboard layout
2. Add real-time status monitoring
3. Implement session tracking
4. Add activity logs
5. Create keyboard shortcuts

**Deliverable:** Functional monitoring dashboard

### Phase 4: QR & Export (Week 3)
1. Implement QR code generation
2. Create config export functionality
3. Add file saving options
4. Test with iOS app (mock if needed)

**Deliverable:** Working QR and export features

### Phase 5: iOS App Foundation (Week 4)
1. Create Xcode project
2. Set up project structure
3. Implement basic UI layouts
4. Create data models
5. Set up Keychain integration

**Deliverable:** iOS app shell with navigation

### Phase 6: iOS Onboarding (Week 5)
1. Build welcome screens
2. Implement QR scanner
3. Create manual setup form
4. Add config import
5. Implement profile management

**Deliverable:** Complete onboarding flow

### Phase 7: iOS Terminal (Week 6)
1. Implement terminal view
2. Add terminal emulator
3. Create keyboard toolbar
4. Add text selection/copy
5. Implement gestures

**Deliverable:** Working terminal UI

### Phase 8: Networking (Week 7)
1. Implement SSH client
2. Add Mosh integration
3. Implement connection management
4. Add auto-reconnection
5. Test with real Mac connection

**Deliverable:** Working SSH/Mosh connections

### Phase 9: Polish & Testing (Week 8)
1. UI/UX refinements
2. Performance optimization
3. Battery usage optimization
4. Comprehensive testing
5. Bug fixes
6. Documentation completion

**Deliverable:** Production-ready apps

---

## Success Criteria

### User Experience
- [ ] Setup takes < 10 minutes on macOS
- [ ] iOS setup via QR code takes < 1 minute
- [ ] Connection establishment < 5 seconds
- [ ] Terminal feels responsive (< 50ms input latency)
- [ ] Handles network changes seamlessly
- [ ] Battery usage < 5% per hour of active use
- [ ] UI is intuitive and beautiful

### Functionality
- [ ] All Tailscale features work
- [ ] SSH connections stable
- [ ] Mosh handles disconnections
- [ ] Claude Code runs perfectly
- [ ] Terminal emulation accurate
- [ ] Copy/paste works reliably
- [ ] Background operation works

### Code Quality
- [ ] Well-structured, modular code
- [ ] Comprehensive error handling
- [ ] Good test coverage
- [ ] Clear documentation
- [ ] Easy to extend
- [ ] Follows platform conventions

### Security
- [ ] All connections encrypted
- [ ] No exposed ports
- [ ] Credentials stored securely
- [ ] No sensitive data in logs
- [ ] Passes security audit

---

## Deployment

### macOS App
- Build for both Intel and Apple Silicon
- Create installer or provide Homebrew formula
- Distribute via GitHub Releases
- Consider Homebrew tap for easy installation

### iOS App
- TestFlight beta initially
- App Store submission after testing
- Prepare marketing materials
- Create App Store screenshots
- Write App Store description

---

## Future Enhancements

### Priority 1 (Next Version)
- Multiple concurrent terminal sessions
- Tab support in iOS app
- File transfer capabilities (scp/sftp)
- Snippet library for common commands
- Connection templates

### Priority 2 (Later)
- iPad-specific UI with split-screen
- Apple Watch companion app
- macOS menu bar app variant
- Notification system for alerts
- Statistics and usage tracking

### Priority 3 (Nice to Have)
- Plugin system for extensions
- Custom terminal themes editor
- Scripting/automation support
- Team/organization features
- Cloud sync for profiles

---

## License

[Choose appropriate license - MIT, Apache 2.0, etc.]

---

## Acknowledgments

- Tailscale for secure networking
- Mosh for resilient connections
- Charm.sh for beautiful TUI libraries
- Claude (Anthropic) for AI-assisted development

---

**Last Updated**: October 17, 2025  
**Version**: 1.0  
**Author**: [Your Name]

---

## Appendix: Reference Resources

### Go/Bubble Tea Resources
- [Bubble Tea Documentation](https://github.com/charmbracelet/bubbletea)
- [Lip Gloss Styling](https://github.com/charmbracelet/lipgloss)
- [Bubbles Components](https://github.com/charmbracelet/bubbles)
- [Example TUI Apps](https://github.com/charmbracelet/bubbletea#examples)

### iOS Development
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui/)
- [Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [Network Framework](https://developer.apple.com/documentation/network)

### Networking & Security
- [Tailscale Documentation](https://tailscale.com/kb/)
- [Mosh Usage](https://mosh.org/)
- [OpenSSH Documentation](https://www.openssh.com/manual.html)

### Terminal Emulation
- [VT100 Specification](https://vt100.net/)
- [ANSI Escape Codes](https://en.wikipedia.org/wiki/ANSI_escape_code)
- [xterm Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html)
