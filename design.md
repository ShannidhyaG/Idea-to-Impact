# Design: Idea to Impact

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Client Application                     │
│                         (React SPA)                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Landing    │  │     App      │  │   Shared     │       │
│  │   Pages      │  │  Interface   │  │  Components  │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              State Management (React Hooks)          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  Gemini Service Layer                │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                │
└────────────────────────────┼────────────────────────────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │   Google Gemini API  │
                  │  (3 Flash Preview)   │
                  └──────────────────────┘
```

### 1.2 Component Hierarchy

```
App
├── CustomCursor
├── Navbar
│   ├── Logo
│   ├── Navigation Links
│   └── Theme Toggle
├── Landing View
│   ├── Hero
│   │   ├── ParticleContainer
│   │   └── CTA Button
│   ├── PictureSlider
│   ├── Tutorial
│   │   └── MockInterface (x3)
│   ├── Features
│   │   └── FeatureCard (x6)
│   └── Footer
└── App View
    ├── AppInterface
    │   ├── AmbientBackground
    │   ├── InputPanel
    │   │   ├── Textarea
    │   │   ├── WordCounter
    │   │   └── ClearButton
    │   ├── ControlPanel
    │   │   ├── ToneSelector
    │   │   ├── PlatformSelector
    │   │   └── TransformButton
    │   └── OutputPanel
    │       ├── PlatformTabs
    │       ├── ContentDisplay (TypewriterText)
    │       └── CopyButton
    └── Footer
```

## 2. Data Models

### 2.1 Type Definitions

```typescript
// Core Types
type PlatformType = 'twitter' | 'linkedin' | 'youtube' | 'instagram' | 'newsletter';
type ToneType = 'professional' | 'casual' | 'enthusiastic' | 'witty' | 'educational' | 'dramatic';
type LengthType = 'short' | 'medium' | 'long';
type ThemeType = 'light' | 'dark';
type ViewType = 'landing' | 'app';

// Transformation Options
interface TransformationOptions {
  tone: ToneType;
  length: LengthType;
  includeEmojis: boolean;
  includeHashtags: boolean;
  formats: PlatformType[];
}

// Transformation Result
interface TransformationResult {
  twitter?: string;
  linkedin?: string;
  youtube?: string;
  instagram?: string;
  newsletter?: string;
}

// Platform Configuration
interface PlatformConfig {
  id: PlatformType;
  label: string;
  icon: React.ReactNode;
  color: string;
  bg: string;
  border: string;
}

// Tone Configuration
interface ToneConfig {
  id: ToneType;
  label: string;
  icon: React.ReactNode;
  color: string;
}

// Feature Item
interface Feature {
  title: string;
  description: string;
  icon: React.ReactNode;
}

// Workflow Step
interface WorkflowStep {
  step: number;
  title: string;
  description: string;
}
```

### 2.2 State Management

```typescript
// App-level State
interface AppState {
  view: ViewType;              // Current view (landing or app)
  theme: ThemeType;             // Current theme
  isMenuOpen: boolean;          // Mobile menu state
}

// AppInterface State
interface AppInterfaceState {
  blogContent: string;                        // User input
  isTransforming: boolean;                    // Loading state
  results: TransformationResult | null;       // AI results
  activeTab: string;                          // Selected platform tab
  selectedFormats: PlatformType[];            // Selected platforms
  selectedTone: ToneType;                     // Selected tone
}

// CustomCursor State
interface CursorState {
  mousePos: { x: number; y: number };
  trail: Array<{ x: number; y: number; id: number }>;
}
```

## 3. Component Design

### 3.1 Core Components

#### App Component
**Purpose**: Root component managing global state and routing between views

**State**:
- `view`: 'landing' | 'app'
- `theme`: 'light' | 'dark'
- `isMenuOpen`: boolean

**Key Features**:
- Lenis smooth scroll initialization
- Theme persistence via localStorage
- View switching between landing and app
- Scroll-to-section navigation

**Lifecycle**:
1. Initialize Lenis on mount
2. Load saved theme from localStorage
3. Apply theme class to document root
4. Clean up Lenis on unmount

#### AppInterface Component
**Purpose**: Main application interface for content transformation

**State**:
- `blogContent`: string (user input)
- `isTransforming`: boolean (loading state)
- `results`: TransformationResult | null
- `activeTab`: string (current platform)

**Form Management**:
- Uses React Hook Form for form state
- Default values: tone='professional', formats=['twitter', 'linkedin']
- Watches: 'formats', 'tone'

**Key Features**:
- Real-time word count
- Sample content injection
- Multi-platform selection
- Tone customization
- AI transformation with loading state
- Tabbed results display
- Clipboard copy functionality

**Workflow**:
1. User pastes content
2. User selects platforms and tone
3. User clicks "Remix Content"
4. Validation check (content not empty)
5. Set loading state
6. Call Gemini service
7. Display results in tabs
8. Enable copy functionality

#### CustomCursor Component
**Purpose**: Enhanced cursor experience with trail effect

**State**:
- `mousePos`: { x, y }
- `trail`: Array of position points (max 15)

**Behavior**:
- Tracks mouse movement
- Creates fading trail particles
- Uses Framer Motion for animations
- Pointer-events: none to avoid interference

#### Hero Component
**Purpose**: Landing page hero section with interactive elements

**Features**:
- Parallax scrolling effects
- Mouse-tracking blob animations
- Particle background
- CTA button to launch app

**Animations**:
- Scroll-based opacity fade
- Mouse-responsive blob movement
- Floating particles
- Morphing blob effects

#### Features Component
**Purpose**: Showcase key features of the application

**Data Source**: FEATURES constant (6 items)

**Features**:
- Grid layout (1/2/3 columns responsive)
- Scroll-triggered animations
- Hover effects (lift and shadow)
- Icon + title + description cards

#### Tutorial Component
**Purpose**: Explain the 3-step workflow

**Data Source**: HOW_IT_WORKS constant

**Features**:
- 3-column grid layout
- Animated mock interfaces for each step
- Step numbers with descriptions
- Interactive UI previews

**Mock Interfaces**:
1. Step 1: Animated input field
2. Step 2: Configuration panel
3. Step 3: Results with copy buttons

### 3.2 Utility Components

#### AmbientBackground
**Purpose**: Decorative gradient blobs for visual appeal

**Implementation**:
- Fixed positioning
- Blur effects (100-120px)
- Color gradients (blue, purple, pink)
- Pointer-events: none

#### ShinyButton
**Purpose**: Reusable animated button with shimmer effect

**Props**:
- onClick: () => void
- disabled: boolean
- children: React.ReactNode
- className: string

**Features**:
- Hover scale animation
- Tap scale animation
- Shimmer effect on hover
- Gradient background
- Disabled state styling

#### TypewriterText
**Purpose**: Animated text reveal effect

**Props**:
- text: string

**Behavior**:
- Character-by-character reveal
- 15ms delay per character
- Preserves whitespace

## 4. Service Layer

### 4.1 Gemini Service

**File**: `services/geminiService.ts`

**Function**: `transformContent(blogContent: string, options: TransformationOptions): Promise<TransformationResult>`

**Implementation**:
```typescript
1. Initialize GoogleGenAI with API key from environment
2. Build platform-specific instructions
3. Construct system prompt with:
   - Requested platforms
   - Tone preference
   - Length preference
   - Emoji/hashtag preferences
   - Platform-specific guidelines
4. Define JSON response schema
5. Call Gemini API with:
   - Model: 'gemini-3-flash-preview'
   - Content: blogContent
   - System instruction
   - Response format: JSON
6. Parse and return structured result
7. Handle errors gracefully
```

**Platform Instructions**:
- **Twitter**: 5-10 tweet thread, hook in first tweet, <280 chars each, hashtags
- **LinkedIn**: Professional tone, 1300-2000 chars, line breaks, 3-5 hashtags, engaging opening
- **YouTube**: Script with intro/main/transitions/CTA, estimated duration
- **Instagram**: Attention-grabbing first line, 150-300 words, line breaks, 10-15 hashtags, emojis
- **Newsletter**: Subject line, structured sections, conversational, PS section

**Error Handling**:
- Catch API errors
- Log to console
- Throw user-friendly error message
- Display toast notification

## 5. Styling Architecture

### 5.1 Tailwind Configuration

**Theme Extensions**:
- Custom colors for brand identity
- Dark mode support via class strategy
- Custom animations (shimmer, morphing-blob, particle)
- Extended spacing and sizing

**Key Classes**:
- `glass-panel`: Glassmorphism effect
- `google-gradient-text`: Brand gradient text
- `flow-text`: Animated gradient text
- `morphing-blob`: Animated blob shapes

### 5.2 Animation System

**Framer Motion Patterns**:
- Page transitions: fade in/out
- Scroll animations: opacity + y-offset
- Hover effects: scale, lift, shadow
- Loading states: spin, pulse
- Stagger animations: sequential reveals

**Custom CSS Animations**:
```css
@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

@keyframes particle {
  0% { transform: translateY(0) scale(1); opacity: 0; }
  50% { opacity: 1; }
  100% { transform: translateY(-100vh) scale(0); opacity: 0; }
}

@keyframes morphing {
  0%, 100% { border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%; }
  50% { border-radius: 30% 60% 70% 40% / 50% 60% 30% 60%; }
}
```

### 5.3 Responsive Design

**Breakpoints**:
- Mobile: < 768px (single column)
- Tablet: 768px - 1024px (2 columns)
- Desktop: > 1024px (3 columns, full layout)

**Responsive Patterns**:
- Grid layouts with responsive columns
- Hidden/visible navigation elements
- Adaptive font sizes
- Touch-friendly tap targets on mobile

## 6. User Flows

### 6.1 First-Time User Flow

```
1. Land on Hero section
   ↓
2. Scroll through Features and Tutorial
   ↓
3. Click "Let's Create" CTA
   ↓
4. Navigate to App Interface
   ↓
5. Click "Use Sample" to see example
   ↓
6. Review sample transformation
   ↓
7. Clear and paste own content
   ↓
8. Select platforms and tone
   ↓
9. Click "Remix Content"
   ↓
10. Review results in tabs
   ↓
11. Copy desired outputs
```

### 6.2 Returning User Flow

```
1. Navigate directly to App Interface
   ↓
2. Paste content
   ↓
3. Select platforms and tone
   ↓
4. Transform
   ↓
5. Copy results
```

### 6.3 Error Recovery Flow

```
1. User attempts transformation without content
   ↓
2. Toast error: "Please enter some content first!"
   ↓
3. User adds content and retries

OR

1. API call fails
   ↓
2. Toast error: "Failed to transform content. Please try again."
   ↓
3. User retries transformation
```

## 7. Performance Optimizations

### 7.1 Code Splitting
- Lazy load landing page components
- Separate bundles for app interface
- Dynamic imports for heavy libraries

### 7.2 Animation Performance
- Use transform and opacity for animations (GPU-accelerated)
- Avoid layout thrashing
- RequestAnimationFrame for smooth scroll
- Debounce mouse tracking

### 7.3 State Management
- Minimize re-renders with React.memo
- Use useCallback for event handlers
- Optimize form watching with React Hook Form

### 7.4 Asset Optimization
- SVG icons (Lucide React)
- No heavy images in critical path
- Lazy load non-critical components

## 8. Accessibility Considerations

### 8.1 Keyboard Navigation
- Tab order follows visual flow
- Focus visible on all interactive elements
- Escape key closes mobile menu
- Enter/Space activates buttons

### 8.2 Screen Readers
- Semantic HTML elements
- ARIA labels for icon buttons
- Alt text for decorative elements
- Live regions for dynamic content

### 8.3 Color Contrast
- WCAG AA compliance for text
- Sufficient contrast in both themes
- Focus indicators clearly visible

### 8.4 Motion Preferences
- Respect prefers-reduced-motion
- Provide option to disable animations
- Essential functionality works without animations

## 9. Security Considerations

### 9.1 API Key Management
- Store in .env.local file
- Never commit to version control
- Use environment variables in production
- Validate key presence before API calls

### 9.2 Input Sanitization
- Validate content length
- Escape special characters
- Prevent XSS attacks
- Rate limit API calls

### 9.3 Data Privacy
- No user data stored on server
- No tracking or analytics (currently)
- All processing client-side
- API calls over HTTPS only

## 10. Testing Strategy

### 10.1 Unit Tests
- Component rendering
- State management logic
- Utility functions
- Form validation

### 10.2 Integration Tests
- Form submission flow
- API service calls
- Navigation between views
- Theme persistence

### 10.3 E2E Tests
- Complete user workflows
- Error scenarios
- Cross-browser compatibility
- Responsive behavior

### 10.4 Manual Testing
- Visual regression testing
- Animation smoothness
- Accessibility audit
- Performance profiling

## 11. Deployment Architecture

### 11.1 Build Process
```
1. npm run build
   ↓
2. Vite bundles application
   ↓
3. Optimize assets
   ↓
4. Generate production build in /dist
```

### 11.2 Environment Configuration
- Development: .env.local
- Production: Environment variables in hosting platform
- Required: GEMINI_API_KEY

### 11.3 Hosting Options
- Vercel (recommended)
- Netlify
- GitHub Pages
- Any static hosting service

### 11.4 CI/CD Pipeline
```
1. Push to main branch
   ↓
2. Run linting and type checking
   ↓
3. Run tests
   ↓
4. Build production bundle
   ↓
5. Deploy to hosting platform
   ↓
6. Run smoke tests
```

## 12. Future Enhancements

### 12.1 Phase 2 Features
- User authentication
- Content history storage
- Custom templates
- Brand voice training

### 12.2 Phase 3 Features
- Direct platform posting
- Scheduling capabilities
- Analytics dashboard
- Team collaboration

### 12.3 Technical Improvements
- Server-side rendering (Next.js)
- Progressive Web App (PWA)
- Offline support
- Real-time collaboration

## 13. Correctness Properties

### 13.1 Input Validation Properties

**Property 1.1**: Content Length Validation
- **Validates**: Requirements 3.1
- **Property**: For all input strings `s`, if `length(s) > 0`, then transformation is enabled
- **Test Strategy**: Generate random strings of varying lengths (0, 1, 100, 10000 chars)
- **Expected**: Button disabled when length = 0, enabled otherwise

**Property 1.2**: Word Count Accuracy
- **Validates**: Requirements 3.1
- **Property**: For all input strings `s`, `displayedWordCount(s) === actualWordCount(s)`
- **Test Strategy**: Generate strings with various whitespace patterns
- **Expected**: Word count matches split(/\s+/).filter(Boolean).length

### 13.2 Platform Selection Properties

**Property 2.1**: Multi-Selection Validity
- **Validates**: Requirements 3.2
- **Property**: For all platform selections `P`, `|P| >= 1` before transformation
- **Test Strategy**: Generate all possible platform combinations
- **Expected**: At least one platform must be selected

**Property 2.2**: Result Completeness
- **Validates**: Requirements 3.2
- **Property**: For all selected platforms `P`, result contains exactly platforms in `P`
- **Test Strategy**: Select various platform combinations, verify result keys
- **Expected**: `keys(result) === P`

### 13.3 Tone Selection Properties

**Property 3.1**: Single Tone Selection
- **Validates**: Requirements 3.3
- **Property**: For all tone selections, exactly one tone is active at any time
- **Test Strategy**: Simulate rapid tone switching
- **Expected**: Only one radio button checked

### 13.4 Transformation Properties

**Property 4.1**: Transformation Idempotency
- **Validates**: Requirements 3.4
- **Property**: For identical inputs `(content, tone, platforms)`, transformation produces consistent structure
- **Test Strategy**: Call transformation multiple times with same inputs
- **Expected**: Result structure remains consistent (keys, format)

**Property 4.2**: Loading State Consistency
- **Validates**: Requirements 3.4
- **Property**: `isTransforming` is true during API call and false after completion/error
- **Test Strategy**: Monitor state during transformation lifecycle
- **Expected**: State transitions: false → true → false

**Property 4.3**: Error Handling Completeness
- **Validates**: Requirements 3.12
- **Property**: For all error conditions, user receives feedback and app remains functional
- **Test Strategy**: Inject API failures, network errors, invalid inputs
- **Expected**: Toast notification shown, no crash, retry possible

### 13.5 UI State Properties

**Property 5.1**: Tab Synchronization
- **Validates**: Requirements 3.5
- **Property**: `activeTab` is always a member of selected platforms
- **Test Strategy**: Generate random platform selections, verify active tab
- **Expected**: `activeTab ∈ selectedPlatforms`

**Property 5.2**: Theme Persistence
- **Validates**: Requirements 3.7
- **Property**: For all theme changes, `localStorage.theme === currentTheme`
- **Test Strategy**: Toggle theme multiple times, check localStorage
- **Expected**: Theme persists across page reloads

**Property 5.3**: Responsive Layout Integrity
- **Validates**: Requirements 3.7
- **Property**: For all viewport sizes, no content overflow or layout breaks
- **Test Strategy**: Test at breakpoints: 320px, 768px, 1024px, 1920px
- **Expected**: All content visible and accessible

### 13.6 Copy Functionality Properties

**Property 6.1**: Clipboard Accuracy
- **Validates**: Requirements 3.6
- **Property**: For all copy operations, `clipboard.content === displayedContent`
- **Test Strategy**: Copy various platform results, verify clipboard
- **Expected**: Exact match between displayed and copied text

**Property 6.2**: Copy Feedback
- **Validates**: Requirements 3.6
- **Property**: For all successful copy operations, user receives confirmation
- **Test Strategy**: Trigger copy, verify toast notification
- **Expected**: Toast appears with "Copied to clipboard" message

### 13.7 Navigation Properties

**Property 7.1**: View State Consistency
- **Validates**: Requirements 3.8
- **Property**: `view` is always either 'landing' or 'app'
- **Test Strategy**: Simulate all navigation actions
- **Expected**: No invalid view states

**Property 7.2**: Scroll Behavior
- **Validates**: Requirements 3.9
- **Property**: For all section links, clicking scrolls to correct section
- **Test Strategy**: Click all navigation links, verify scroll position
- **Expected**: Target section in viewport after scroll

### 13.8 Performance Properties

**Property 8.1**: Animation Frame Rate
- **Validates**: Requirements 3.10
- **Property**: For all animations, frame rate >= 50fps on modern devices
- **Test Strategy**: Monitor FPS during animations
- **Expected**: Smooth animations without jank

**Property 8.2**: Transformation Timeout
- **Validates**: Requirements 3.4
- **Property**: For all transformations, completion time < 10 seconds
- **Test Strategy**: Measure time from click to result display
- **Expected**: Duration < 10000ms

### 13.9 Data Integrity Properties

**Property 9.1**: Content Preservation
- **Validates**: Requirements 3.12
- **Property**: For all transformations, original content remains unchanged
- **Test Strategy**: Transform content, verify input field unchanged
- **Expected**: `inputContent === originalContent` after transformation

**Property 9.2**: Result Structure Validity
- **Validates**: Requirements 3.5
- **Property**: For all results, each platform value is a non-empty string
- **Test Strategy**: Generate results, validate structure
- **Expected**: `typeof result[platform] === 'string' && result[platform].length > 0`

## 14. Property-Based Testing Framework

### 14.1 Testing Tools
- **Framework**: Vitest
- **PBT Library**: fast-check
- **Assertion Library**: Vitest expect

### 14.2 Test Organization
```
tests/
├── unit/
│   ├── components/
│   ├── services/
│   └── utils/
├── integration/
│   ├── transformation-flow.test.ts
│   └── navigation-flow.test.ts
└── properties/
    ├── input-validation.property.test.ts
    ├── platform-selection.property.test.ts
    ├── transformation.property.test.ts
    └── ui-state.property.test.ts
```

### 14.3 Generator Strategies

**Content Generator**:
```typescript
const contentArbitrary = fc.string({ minLength: 0, maxLength: 10000 });
const wordCountArbitrary = fc.array(fc.string(), { minLength: 0, maxLength: 1000 });
```

**Platform Generator**:
```typescript
const platformArbitrary = fc.subarray(
  ['twitter', 'linkedin', 'instagram', 'facebook'],
  { minLength: 1, maxLength: 4 }
);
```

**Tone Generator**:
```typescript
const toneArbitrary = fc.constantFrom(
  'professional', 'casual', 'witty', 'educational', 'dramatic'
);
```

### 14.4 Test Execution Strategy
1. Run unit tests first (fast feedback)
2. Run property tests (comprehensive coverage)
3. Run integration tests (workflow validation)
4. Generate coverage report
5. Fail build if coverage < 80%
