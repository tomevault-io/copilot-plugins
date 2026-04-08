## securegen

> ![GitLab CI](https://img.shields.io/gitlab/pipeline/your-username/securegen?style=flat-square&logo=gitlab)

# SecureGen - Thai Smart Password Manager 🇹🇭

![GitLab CI](https://img.shields.io/gitlab/pipeline/your-username/securegen?style=flat-square&logo=gitlab)
![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)
![React](https://img.shields.io/badge/React-19-blue?style=flat-square&logo=react)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?style=flat-square&logo=typescript)

**SecureGen** is a client-side password generator and manager designed specifically for Thai users. It leverages the "Thai-Shift" keyboard logic to convert memorable Thai phrases into complex English passwords, alongside AI-powered mnemonic generation.

**Live Demo:** [Link to your GitLab Pages]

---

## 📚 Documentation (เอกสารประกอบ)

*   **[📖 User Manual (คู่มือการใช้งาน)](App.tsx)** - Built inside the App (Click the Help/Book icon).
*   **[📘 Beginner Developer Manual (คู่มือนักพัฒนาฉบับเริ่มต้น)](DevManual.md)** - **New to coding? Start here!** หากคุณเป็นนักพัฒนาหน้าใหม่ อ่านหน้านี้ก่อนครับ.
*   **[🛠️ Technical Design Spec](designspec.md)** - Deep dive into architecture and logic.
*   **[🧪 Test Specification](TestSpec.md)** - How to test the application.

---

## 📋 Requirements & Design Specification

### Overview
SecureGen is a Progressive Web Application (PWA) built with React 19 and TypeScript 5. It provides Thai users with a secure, client-side password generation and management system that leverages Thai keyboard input patterns to create memorable yet complex passwords.

### Core Requirements

#### 1. Thai-Shift Password Generator
- **Requirement**: Convert Thai phrases into complex English passwords using QWERTY key mapping
- **Implementation**: Thai-to-English character mapping based on "Thai-Shift" keyboard logic
- **Example**: Thai phrase "สวัสดี" converts to corresponding QWERTY keys
- **Use Case**: Users can remember Thai phrases instead of complex English passwords

#### 2. AI-Powered Creative Mode
- **Requirement**: Generate secure passwords from context or mnemonics using AI
- **Technology**: Google Gemini API integration
- **Privacy Model**: BYOK (Bring Your Own Key) - Users provide their own API keys
- **Stateless**: No password data stored server-side; only generation prompts are sent
- **Feature**: Users can input context/description and receive AI-generated passwords

#### 3. Memory Test Feature
- **Requirement**: Built-in tool to verify users can remember generated passwords
- **Purpose**: Ensure password memorability before deployment
- **UX**: Interactive test interface allowing password recall verification

#### 4. Client-Side Architecture
- **Requirement**: All processing happens in browser; zero-knowledge architecture
- **Data Storage**: Passwords stored only in browser RAM
- **Data Clearing**: Automatic clearing when tab/browser session closes
- **Security**: No data transmission to servers (except stateless AI prompts)

#### 5. Encrypted Export/Backup
- **Requirement**: Export credentials with AES encryption
- **Purpose**: Secure backup capability for users
- **Format**: Encrypted credential bundles for local storage or safe transfer
- **Decryption**: User-controlled decryption with password/key

#### 6. Progressive Web App (PWA)
- **Requirement**: Installable on mobile and desktop devices
- **Features**: Offline capability, app-like experience, home screen installation
- **Manifest**: PWA manifest configuration for various platforms
- **Responsive**: Works seamlessly on all screen sizes

#### 7. Crypto Wallet Mode (BTC Edition)
- **Requirement**: Support Seed Phrase/Mnemonic generation for cryptocurrency wallets
- **Mnemonic Support**: Generate 12 or 24-word seed phrases following BIP39 standards
- **Wallet Address Tracking**: Store and manage public addresses paired with private keys
- **Safety Warnings**: Display risk alerts for blockchain key management
- **Crypto-Specific UI**: Dark gold/orange theme with distinct visual identity

### Technical Architecture

#### Tech Stack
- **Frontend Framework**: React 19
- **Language**: TypeScript 5
- **API Integration**: Google Gemini API (optional, for AI features)
- **Encryption**: AES-256 (for export functionality)
- **Build Tool**: Vite (assumed based on env file pattern)
- **Deployment**: GitLab Pages with GitLab CI/CD
- **Icons**: Lucide-React for UI elements
- **Crypto Utilities**: Crypto-JS for AES encryption simulation

#### Project Structure
```
securegen/
├── public/             # Static assets (icons, manifest)
│   ├── manifest.json   # PWA manifest
│   └── icons/          # App icons for various sizes
├── src/
│   ├── services/       # Core business logic
│   │   ├── ThaiMapper.ts           # Thai-to-English character mapping
│   │   ├── CryptoService.ts        # AES encryption/decryption
│   │   ├── PasswordValidator.ts    # Password strength evaluation
│   │   ├── GeminiService.ts        # AI prompt handling
│   │   ├── MnemonicGenerator.ts    # Seed phrase generation (BIP39 simulation)
│   │   └── WalletManager.ts        # Crypto wallet operations
│   ├── components/     # React components
│   │   ├── PasswordGenerator.tsx
│   │   ├── CreativeMode.tsx
│   │   ├── MemoryTest.tsx
│   │   ├── ExportImport.tsx
│   │   ├── CryptoWallet.tsx        # Crypto wallet mode
│   │   ├── MnemonicDisplay.tsx     # Seed phrase display
│   │   ├── AddressManager.tsx      # Wallet address tracking
│   │   └── Help.tsx
│   ├── types.ts        # TypeScript interfaces and types
│   ├── App.tsx         # Main component with routing/state
│   └── index.tsx       # Entry point
├── .gitlab-ci.yml      # GitLab CI/CD Configuration
├── package.json        # Dependencies & Scripts
├── vite.config.ts      # Vite configuration
├── tsconfig.json       # TypeScript configuration
└── README.md           # User documentation
```

#### Key Services

**ThaiMapper Service**
- Maps Thai characters to QWERTY keyboard positions
- Supports Thai-Shift keyboard layout
- Handles tone marks and special characters

**CryptoService**
- AES-256 encryption for export functionality
- Secure key derivation
- Encryption/decryption with configurable strength

**GeminiService**
- Integration with Google Gemini API
- BYOK pattern implementation
- Stateless prompt transmission
- Error handling for API failures

**PasswordValidator Service**
- Password strength calculation
- Complexity requirements validation
- Memorability assessment

**MnemonicGenerator Service**
- Generate BIP39-compliant seed phrases (12 or 24 words)
- Word list management for crypto mnemonics
- Validation of generated seeds

**WalletManager Service**
- Store and retrieve wallet addresses
- Associate addresses with seed phrases
- Export wallet information securely

### Data Flow

1. **Thai-Shift Generation Flow**
   - User inputs Thai phrase
   - ThaiMapper converts to QWERTY positions
   - Password generated with complexity options
   - Display to user for approval

2. **AI Creative Mode Flow**
   - User provides context/description
   - Prompt sent to Gemini API (via user's API key)
   - Generated password received
   - User can accept or regenerate

3. **Memory Test Flow**
   - User views generated password (optional memorization time)
   - Test begins with hidden password
   - User attempts to recall and enter password
   - System validates recall accuracy
   - Feedback provided to user

4. **Export Flow**
   - User selects credentials to export
   - System applies AES encryption
   - Encrypted file downloaded to user's device
   - User can backup or securely share

5. **Crypto Seed Generation Flow**
   - User enters Crypto Wallet Mode
   - Safety warning displayed prominently
   - User selects 12 or 24-word mnemonic
   - Seed phrase generated and displayed
   - User can opt for AI hint generation
   - User enters wallet addresses for tracking
   - Everything encrypted on export

### Security Considerations

#### Privacy Model
- **Client-Side Processing**: All password generation happens in-browser
- **No Storage**: No passwords stored on servers
- **Session Cleanup**: RAM cleared on session end
- **Transparent Data Flow**: Only API keys and generation prompts sent to Gemini API
- **Crypto Isolation**: Seed phrases never leave device except via encrypted export

#### Encryption Standards
- **AES-256** for export encryption
- **PBKDF2** or **bcrypt** for key derivation (from user passwords)
- **Random salt generation** for each export
- **Secure randomness** for seed phrase generation

#### Authentication & Authorization
- **No user accounts required** for basic functionality
- **Optional**: API key storage (user's choice, local storage only)
- **Ephemeral data**: No persistent user authentication
- **Master Password**: Required for export encryption strength

#### Risk Warnings & Safeguards
- **Crypto Mode Warning**: Display safety alert when entering crypto features
- **Seed Phrase Backup Alert**: Remind users to safely store backups
- **No Recovery Option**: Clearly state that lost seeds cannot be recovered
- **Private Key Warnings**: Educate users on risks of key exposure

### Environment Variables
```
VITE_API_KEY=optional_default_gemini_api_key
```
- Optional for development
- Users can enter their own keys in UI
- Not required for Thai-Shift generator, export, or crypto features

### UI/UX Specifications

#### Themes & Styling
- **Default Theme**: Clean, modern interface for password management
- **Crypto Theme**: Dark gold/orange accents for wallet mode to distinguish from regular passwords
- **Accessibility**: High contrast ratios, keyboard navigation support
- **Responsive Design**: Mobile-first approach with tablet/desktop optimization

#### Icons & Components
- **Icon Library**: Lucide-React for consistent, modern icons
- **Crypto Icons**: Wallet, Coins, ShieldAlert for blockchain features
- **Visual Hierarchy**: Clear distinction between modes

#### Component Behavior
- **Modal Alerts**: Safety warnings for crypto operations
- **Copy-to-Clipboard**: One-click copying for generated passwords/seeds
- **Progress Indicators**: Feedback for encryption/generation operations
- **Tooltips**: Contextual help for complex operations

### Feature Requirements Summary

| Feature | Status | Priority | Notes |
|---------|--------|----------|-------|
| Thai-Shift Generator | Core | High | Primary feature for Thai users |
| AI Creative Mode | Enhanced | Medium | Optional with user's API key |
| Memory Test | Enhanced | Medium | Improves password adoption |
| Encrypted Export | Core | High | Critical for backup/security |
| PWA Installation | Enhanced | Medium | Mobile accessibility |
| Zero-Knowledge Arch | Core | High | Security foundation |
| Crypto Seed Generation | Core | High | BTC wallet support |
| Wallet Address Tracking | Core | High | Address management |
| Crypto Safety Warnings | Core | High | User protection |
| Mnemonic AI Hints | Enhanced | Medium | UX improvement for crypto |

### Development Workflow

1. **Feature Development**: Implement feature with TypeScript
2. **Component Testing**: Unit tests for services and components
3. **Integration Testing**: End-to-end password generation and crypto workflows
4. **Manual Testing**: Memory test, export/import cycles, browser compatibility, crypto operations
5. **GitLab CI/CD**: Automated testing and deployment to GitLab Pages

### Performance Considerations

- **Password Generation**: < 100ms for Thai-Shift conversion
- **AI API Calls**: Network dependent, user-initiated
- **Export Encryption**: < 1s for typical credential sets
- **Seed Generation**: < 50ms for mnemonic creation
- **Memory Usage**: Minimal, cleared on session end
- **Bundle Size**: Optimize React and dependencies for fast loading

### Browser Compatibility
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+
- Mobile browsers (iOS Safari, Chrome Mobile)
- PWA support on all modern browsers

### Accessibility Requirements
- WCAG 2.1 AA compliance target
- Keyboard navigation support
- Screen reader friendly
- Color contrast requirements met (especially for crypto warning colors)
- Thai language support in all UI elements
- Clear warning text for crypto operations

---

## ✨ Key Features (คุณสมบัติหลัก)

*   **🇹🇭 Thai-Shift Generator:** Convert Thai phrases (e.g., "สวัสดี") into complex strings based on QWERTY key mapping.
*   **🤖 AI Creative Mode:** Use Google Gemini API to generate secure passwords from context/mnemonics.
*   **🧠 Memory Test:** Built-in tool to test if you can remember the generated password before using it.
*   **🔒 Client-Side Only:** Runs entirely in the browser. No data is ever sent to a server (except for AI generation prompts, which are stateless).
*   **💾 Encrypted Export:** Export your credentials using AES encryption for backup.
*   **📱 PWA Ready:** Installable on mobile and desktop.

---

## 🚀 Getting Started (การติดตั้งและใช้งาน)

### Prerequisites
*   Node.js (v18 or higher)
*   npm or yarn

### Installation

1.  **Clone the repository**
    ```bash
    git clone https://gitlab.com/your-username/securegen.git
    cd securegen
    ```

2.  **Install dependencies**
    ```bash
    npm install
    ```

3.  **Set up Environment Variables (Optional)**
    If you want to use the AI features locally without entering the key in the UI every time, create a `.env` file:
    ```env
    VITE_API_KEY=your_gemini_api_key_here
    ```

4.  **Run Development Server**
    ```bash
    npm run dev
    ```
    Open `http://localhost:3000` in your browser.

---

## 🏗️ Project Structure

```
securegen/
├── public/             # Static assets (icons, manifest)
├── src/
│   ├── services/       # Core logic (Crypto, Thai Mapping)
│   ├── types.ts        # TypeScript interfaces
│   ├── App.tsx         # Main Component
│   └── index.tsx       # Entry point
├── .gitlab-ci.yml      # GitLab CI/CD Configuration
├── package.json        # Dependencies & Scripts
└── README.md           # Documentation
```

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to submit Merge Requests and report issues.

---

## 🛡️ Privacy & Security

*   **Zero-Knowledge Architecture:** We do not store your passwords. Everything is stored in your browser's RAM and cleared when the tab is closed.
*   **BYOK (Bring Your Own Key):** For AI features, users provide their own Gemini API keys for maximum privacy.

---

## 📞 Contact & Credits

**Presented by:** SPADA Service (Nextwaver.Net Co., Ltd.)  
**Developer:** Mr. Chatchai Limprasertsiri  
**Email:** chatchai@nextwaver.com

Licensed under the [MIT License](LICENSE).

=============================


# SecureGen - Thai Smart Password & Wallet Manager
## Design & Requirement Specification (v1.1 - Crypto Update)

---

## 1. Project Overview
SecureGen (BTC Edition) เพิ่มขีดความสามารถในการจัดการ Key สำหรับ Cryptocurrency โดยเฉพาะ BTC E-Wallet เพื่อให้ผู้ใช้สามารถสร้างและจัดเก็บ Seed Phrase/Private Key ได้อย่างปลอดภัยภายใต้สถาปัตยกรรม Zero-Knowledge

### 1.1 Core Value Proposition (Crypto Addition)
1.  **Seed Phrase Generation:** สร้าง Mnemonic 12/24 คำ ตามมาตรฐานความปลอดภัย
2.  **Wallet Address Tracking:** จัดเก็บเลขกระเป๋า (Public Address) คู่กับกุญแจส่วนตัว
3.  **Safety First UI:** มีระบบแจ้งเตือนความเสี่ยงในการจัดการกุญแจ Blockchain

---

## 2. Technical Architecture
*   **Encryption:** AES-256 via Crypto-JS สำหรับการ Export ข้อมูล
*   **Mnemonic Logic:** (Simulation) ระบบสุ่มคำศัพท์จากฐานข้อมูลที่กำหนด เพื่อใช้เป็น Seed
*   **Privacy:** ข้อมูลทั้งหมดไม่เคยออกจากเครื่องผู้ใช้ (ยกเว้นการส่ง Prompt ไป AI ซึ่งไม่มีข้อมูลส่วนตัว)

---

## 3. Functional Requirements

### 3.1 Crypto Wallet Mode
*   **Mnemonic Generator:** เลือกได้ว่าจะสร้าง 12 หรือ 24 คำ
*   **Address Management:** ฟิลด์พิเศษสำหรับเก็บเลขกระเป๋า เพื่อความสะดวกในการ Copy ไปใช้งาน
*   **Safety Warning:** แสดงข้อความเตือนทุกครั้งที่เข้าโหมด Crypto

### 3.2 AI Creative (Crypto Version)
*   **Hint Optimization:** AI ช่วยสร้างคำใบ้สำหรับ Seed Phrase ที่ผู้ใช้เข้าใจคนเดียว แต่คนอื่นเดาไม่ออก

---

## 4. UI/UX Specifications
*   **Theme:** Dark Gold/Orange accents สำหรับส่วนที่เป็น Crypto เพื่อให้แตกต่างจากรหัสผ่านทั่วไป
*   **Icons:** ใช้ Lucide-React (Wallet, Coins, ShieldAlert)

---

## 5. Security Protocols
1.  **No Persistence:** ข้อมูลจะหายไปทันทีเมื่อ Refresh หรือปิด Tab
2.  **Encrypted Export:** ข้อมูลที่ Export จะต้องมี Master Password ที่มีความซับซ้อนเท่านั้น

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chatchailim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chatchailim)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
