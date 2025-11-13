# Product Requirements Document: Whole Heart Publishing Batch Generator

## Document Information
- **Product Name:** Whole Heart Publishing Batch Generator
- **Client:** Whole Heart Publishing
- **Version:** 2.0
- **Last Updated:** November 12, 2025
- **Document Owner:** [To be assigned]
- **Status:** Draft

---

## 1. Executive Summary

The Whole Heart Publishing Batch Generator is a web-based application designed to streamline high-volume AI image generation through OpenAI's DALL-E 3 model. This tool addresses the need for efficient, large-scale image creation by enabling users to submit 50-100 prompts concurrently and receive generated images as downloadable JPEG archives.

### Key Value Proposition
- **Efficiency:** Generate multiple images simultaneously rather than one-by-one
- **Convenience:** Bulk prompt entry via text area (100 prompt limit)
- **Control:** Pause, resume, or cancel batch operations
- **Organization:** Consolidated ZIP downloads and persistent JPEG image storage
- **Quality:** DALL-E 3 model support with customizable style and aspect ratio options

### Competitive Reference
[DALL-E Bulk Image Generator](https://dall-ebulkimagegenerator.com/)

### Client Information
- **Domain:** Subdomain to wholeheartpublishing.com
- **Current Hosting:** Squarespace

---

## 2. Product Objectives

### Primary Goals
1. Enable high-volume image generation (50-100 prompts per batch)
2. Provide seamless integration with OpenAI's DALL-E 3 API
3. Deliver user-friendly bulk JPEG download capabilities
4. Ensure reliable batch job management and error handling
5. Support DALL-E 3 specific parameters (style and aspect ratio)

### Success Metrics
- **Throughput:** Successfully process batches of 50-100 prompts
- **Reliability:** 95%+ successful image generation rate (excluding content policy violations)
- **User Satisfaction:** Reduce time-to-download by 80% compared to manual individual generation
- **Error Recovery:** 100% of partial batches recoverable after pause/cancel operations

---

## 3. User Personas

### Primary Persona: Content Creator
- **Background:** Marketing professionals, graphic designers, social media managers
- **Needs:** Generate multiple variations of promotional images, product mockups, or social media assets
- **Pain Points:** Manual one-by-one generation is time-consuming and inefficient
- **Technical Proficiency:** Moderate; comfortable with CSV files and basic API concepts

### Secondary Persona: AI Enthusiast/Researcher
- **Background:** AI researchers, developers experimenting with image generation
- **Needs:** Test multiple prompts systematically, compare outputs across parameter variations
- **Pain Points:** Need programmatic access but prefer GUI for quick experiments
- **Technical Proficiency:** High; may have API keys and understand technical limitations

---

## 4. Assumptions and Prerequisites

### User Requirements
1. **Active OpenAI Account**
   - Account must be adequately funded (automatic funding enabled OR manual balance maintenance)
   - Valid, active API key
   - Organization verified within OpenAI for image generation endpoint access

### Technical Assumptions
- Users have modern web browsers (Chrome, Firefox, Safari, Edge - latest 2 versions)
- Stable internet connection for API communications
- Local storage availability for image persistence

### Constraints
- OpenAI API does not support true bulk generation; sequential API calls required
- API rate limits and quotas apply per user's OpenAI account tier
- Content policy violations are handled at the API level

---

## 5. Features and Requirements

### 5.1 Bulk Prompt Submission

#### 5.1.1 Manual Text Input
**Priority:** P0 (Must Have)

**Description:** Users can manually enter prompts line-by-line into a dedicated text area.

**Acceptance Criteria:**
- Text area supports up to 100 prompts
- Each prompt is entered on a new line
- Line numbers are displayed for easy reference
- Real-time count of entered prompts is visible
- UI prevents or warns when exceeding 100 prompts

**User Flow:**
1. User navigates to prompt input interface
2. User types or pastes prompts into text area (one per line)
3. System displays prompt count (e.g., "25/100 prompts")
4. User selects image generation parameters (background, output format, size)
5. User clicks "Generate Images" button
6. System navigates to progress page
7. Progress bar updates in real-time as each image generates
8. Upon completion, system navigates to results page
9. User reviews generated images in gallery view
10. User can:
    - Download individual images by clicking download button on each image
    - Select multiple images using checkboxes
    - Click "Select All" to select entire batch
    - Click "Download Selected" to download as ZIP file
11. System packages selected images into ZIP archive
12. ZIP file downloads to user's local machine

---

### 5.2 Image Generation Parameters

#### 5.2.1 Style Selection
**Priority:** P0 (Must Have)

**Options:**
- Vivid (default) - More hyper-real and dramatic images
- Natural - More natural, less hyper-real looking images

**UI Component:** Radio buttons or dropdown

**API Mapping:**
- User selects "Vivid" → `"style": "vivid"` in API request
- User selects "Natural" → `"style": "natural"` in API request

---

#### 5.2.2 Image Size Selection
**Priority:** P0 (Must Have)

**Options:**
- **(9:16)** - Portrait orientation
- **(1:1)** - Square orientation

**UI Component:** Radio buttons or dropdown

**API Mapping:**
- User selects **(9:16)** → `"size": "1024x1792"` in API request
- User selects **(1:1)** → `"size": "1024x1024"` in API request (default)

**Note:** These are the only two sizes supported by DALL-E 3 for this application.

---

### 5.3 API Integration

#### 5.3.1 OpenAI DALL-E 3 API Connection
**Priority:** P0 (Must Have)

**Endpoint:** `https://api.openai.com/v1/images/generations`

**Model:** `dall-e-3`

**Request Specification:**
```json
{
  "model": "dall-e-3",
  "prompt": "[User-submitted prompt text]",
  "style": "vivid|natural",
  "n": 1,
  "size": "1024x1024|1024x1792"
}
```

**Request Parameters:**
| Parameter | Type | Required | Value | Notes |
|-----------|------|----------|-------|-------|
| model | string | Yes | "dall-e-3" | Hardcoded |
| prompt | string | Yes | User input | Variable |
| style | string | Yes | "vivid" or "natural" | User-selected, default "vivid" |
| n | integer | Yes | 1 | Hardcoded |
| size | string | Yes | "1024x1024" or "1024x1792" | User-selected, default "1024x1024" |

**Response Format:**
```json
{
  "created": 1762984113,
  "data": [
    {
      "revised_prompt": "An East Asian man seated in a comfortable gaming chair...",
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-.../img-xxx.png?st=..."
    }
  ]
}
```

**Key Response Fields:**
- `created`: Unix timestamp
- `revised_prompt`: OpenAI's enhanced/safety-filtered version of the original prompt
- `url`: Temporary URL to the generated image (expires in 60 minutes)

**Acceptance Criteria:**
- Sequential API calls (one per prompt, not parallel)
- Each API call includes user's API key in Authorization header
- Response includes temporary image URL (valid for 60 minutes)
- System handles rate limiting gracefully
- Timeout set to 30 seconds per request
- Retry mechanism for transient errors (3 attempts with exponential backoff)

---

#### 5.3.2 URL-to-JPEG Conversion and Storage (FR-14)
**Priority:** P0 (Must Have)

**Description:** Convert temporary OpenAI image URLs to permanent JPEG files stored in Supabase Storage.

**Critical Requirement:** OpenAI URLs expire after 60 minutes, so images must be fetched, converted, and stored immediately upon generation.

**Processing Requirements:**

1. **Fetch Binary Image**
   - Edge Function fetches the binary image data from the OpenAI-provided URL
   - Must complete within URL validity period (60 minutes, but should be immediate)

2. **Convert to JPEG Format**
   - Conversion performed **server-side in Supabase Edge Function**
   - Preserve original image resolution
   - Use configurable JPEG quality setting (recommended: 80-95%)
   - No client-side conversion

3. **Storage Strategy**
   - Option A: Store in memory for immediate ZIP creation
   - Option B: Save to Supabase Storage bucket (e.g., `images/temp/{jobId}/{index}.jpg`)
   - Recommended: Option B for persistence and reliability

4. **File Naming Convention**
   - Format: `{jobId}_{index}.jpg` or zero-padded `{jobId}_001.jpg`
   - Must maintain association with originating prompt
   - Sequential numbering for batch organization

5. **Database Metadata**
   - Store in `images` table:
     - `batch_id`: Links to batch job
     - `prompt`: Original user prompt
     - `revised_prompt`: OpenAI's revised prompt
     - `storage_url`: Supabase Storage public URL
     - `status`: success/failed/blocked
     - `error_message`: If applicable
     - `created_at`: Timestamp

**Acceptance Criteria:**
- Successfully fetch and convert 100% of valid OpenAI URLs
- JPEG quality is consistent across all images
- No image data loss during conversion
- Storage URLs are permanent and accessible
- Failed conversions are logged with specific error messages
- Conversion completes within 5 seconds per image

---

### 5.4 Batch Job Management

#### 5.4.1 Job Controls
**Priority:** P0 (Must Have)

**Description:** Users can control batch operations through pause, resume, and cancel actions.

**Pause Functionality:**
- Waits for current API call to complete
- Returns all completed prompts/images
- Maintains queue state for resume
- Updates progress bar to show paused state

**Resume Functionality:**
- Continues from prompt immediately after last completed
- Maintains all previous settings
- Updates progress bar to show active state

**Cancel Functionality:**
- Immediately halts new API calls
- Completes any in-flight API requests
- All completed images remain available for download
- Clears queue state

**Acceptance Criteria:**
- Pause button becomes resume button when clicked
- Cancel provides confirmation dialog
- No data loss for completed images during any operation
- State transitions are clearly communicated to user

---

#### 5.4.2 Progress Tracking
**Priority:** P0 (Must Have)

**Description:** Visual progress indicator showing batch job completion status.

**Acceptance Criteria:**
- Progress bar shows percentage complete (e.g., "35/100 prompts completed")
- Real-time updates as each image generates
- Displays estimated time remaining
- Shows current prompt being processed
- Error count displayed separately (e.g., "35 completed, 2 failed")

**UI Components:**
- Linear progress bar
- Numerical counter (X/Y completed)
- Status text (Processing, Paused, Completed, Cancelled)
- Error indicator

---

### 5.5 Image Visualization and Download

#### 5.5.1 Image Preview
**Priority:** P0 (Must Have)

**Description:** Generated images are displayed in a gallery view within the application.

**Acceptance Criteria:**
- Thumbnail preview of all generated images
- Grid or list view options
- Click to view full-size image
- Display associated prompt text with each image
- Indicate failed/blocked prompts with error icon

**UI Layout:**
```
[Thumbnail] [Thumbnail] [Thumbnail]
Prompt 1    Prompt 2    Prompt 3
[Download]  [Download]  [Download]
```

---

#### 5.5.2 Individual Image Download
**Priority:** P0 (Must Have)

**Description:** Users can download images individually in JPEG format.

**Acceptance Criteria:**
- Download button available for each image
- File naming convention: `image_{prompt_number}_{timestamp}.jpg`
- JPEG format only (PNG not supported)
- Download triggers browser's native download
- Image quality preserved from conversion process

---

#### 5.5.3 Bulk Download
**Priority:** P0 (Must Have)

**Description:** Users can select multiple or all images for download as a single ZIP archive containing JPEG files.

**Acceptance Criteria:**
- "Select All" checkbox available
- Individual image checkboxes for selective download
- "Download Selected" button active when 2+ images selected
- ZIP file naming: `bulk_images_{timestamp}.zip`
- ZIP contains all selected JPEG images with descriptive filenames
- Progress indicator during ZIP creation
- Fallback to individual downloads if ZIP creation fails
- All images in ZIP are JPEG format

---

### 5.6 Image Persistence and Management

#### 5.6.1 Storage
**Priority:** P1 (Should Have)

**Description:** Generated images are stored for future access, re-download, or reuse.

**Acceptance Criteria:**
- Images persist across browser sessions
- User can navigate away and return to view previous generations
- Storage organized by batch/generation session
- Display generation date/time for each batch

---

#### 5.6.2 Image Deletion
**Priority:** P1 (Should Have)

**Description:** Users can delete stored images to manage storage.

**Acceptance Criteria:**
- Delete individual images
- Delete entire batch
- Confirmation dialog before deletion
- Deletion is permanent (no undo)
- Storage space updates after deletion

---

## 6. Error Handling and Edge Cases

### 6.1 Input Validation Errors

#### 6.1.1 Empty Prompts
**Scenario:** Prompt line is empty or contains only whitespace

**Expected Behavior:**
- Skip empty prompt during processing
- Do not send API call for empty prompt
- Log as skipped in results
- Continue with next valid prompt

---

#### 6.1.2 Text Area Limit Exceeded
**Scenario:** User types >100 lines in text area

**Expected Behavior:**
- UI prevents input after 100 lines, OR
- Backend truncates to first 100 prompts with notification

---

#### 6.1.3 Two Prompts in One Line
**Scenario:** User enters two distinct prompts within a single line of text

**Expected Behavior:**
- UI visually indicates error by underlining the text with a red line
- Error message displayed: "Each prompt must be on a separate line"
- Prevent generation until error is corrected
- Enforce one prompt per line rule

---

### 6.2 API and Authentication Errors

#### 6.2.1 Invalid/Expired API Key
**Scenario:** User's API key is not accepted by OpenAI

**Expected Behavior:**
- All generation attempts fail immediately
- Display error: "Invalid API key. Please update your API key in settings."
- Prevent batch job from starting
- Provide link to API key settings

---

#### 6.2.2 Insufficient Funds
**Scenario:** OpenAI account runs out of credits during batch

**Expected Behavior:**
- API returns payment/billing error
- Pause batch job automatically
- Display error: "Insufficient funds in OpenAI account. Please add credits to continue."
- Show which prompts were completed successfully
- Allow user to resume after funding account

---

#### 6.2.3 Permissions Error
**Scenario:** API key lacks image generation permissions

**Expected Behavior:**
- API returns 403 Forbidden or similar
- Display error: "Your API key does not have permission to access image generation. Please verify your organization in OpenAI."
- Provide link to OpenAI verification documentation

---

#### 6.2.4 API Timeouts/Server Errors
**Scenario:** OpenAI API is temporarily unavailable or request times out

**Expected Behavior:**
- Implement retry mechanism (3 attempts with exponential backoff)
- For 5xx errors: retry up to 3 times
- If all retries fail: mark prompt as failed
- Continue batch with next prompt
- Display warning: "Some prompts failed due to API issues. Failed prompts: [list]"

---

#### 6.2.5 Content Policy Violation
**Scenario:** Prompt violates OpenAI's content policy

**Expected Behavior:**
- API returns content violation error
- Skip prompt, mark as blocked
- Log reason: "Content policy violation"
- Continue batch with next prompt
- Display warning icon next to failed prompt
- Batch continues uninterrupted

---

### 6.3 Generation and Output Errors

#### 6.3.1 URL Fetch or JPEG Conversion Failure
**Scenario:** OpenAI returns valid URL but Edge Function fails to fetch image or convert to JPEG

**Expected Behavior:**
- Image fails to generate preview
- Display error placeholder for that image
- Mark as failed in results with specific error message
- Log error: "Failed to fetch/convert image from URL"
- Allow user to retry individual prompt
- Continue batch with next prompt

**Possible Causes:**
- OpenAI URL expired before fetch (unlikely if immediate)
- Network timeout during fetch
- Image conversion library error
- Insufficient memory for conversion

---

#### 6.3.2 Zero Images Generated
**Scenario:** Entire batch fails (e.g., all prompts blocked or API key error)

**Expected Behavior:**
- Results page displays: "No images were generated."
- Provide explanation: "All prompts failed due to: [reason]"
- Offer troubleshooting steps
- Allow user to review and retry prompts

---

#### 6.3.3 Bulk Download Failure
**Scenario:** ZIP creation fails (memory/I/O error)

**Expected Behavior:**
- Display error: "Bulk download failed. Please try downloading images individually."
- Provide "Download All Individually" button as fallback
- Log error for debugging

---

### 6.4 User Action Edge Cases

#### 6.4.1 Pause During Active Request
**Scenario:** User pauses while API call is in transit

**Expected Behavior:**
- System waits for current API call to complete
- Result (success or failure) is recorded
- Pause takes effect before next API call
- Progress bar shows accurate count

---

#### 6.4.2 Cancel During Completion
**Scenario:** Multiple API calls returning concurrently when user cancels

**Expected Behavior:**
- Store all results received before cancellation fully processes
- Halt all subsequent API calls in queue
- All received images available for download
- Display count: "X of Y images completed before cancellation"

---

#### 6.4.3 No Parameters Selected
**Scenario:** User uploads prompts but doesn't select any parameters

**Expected Behavior:**
- Allow generation to proceed with default values
- Use defaults: style=vivid, size=1024x1024 (1:1)
- Do not block generation
- Display selected defaults in UI before generation

---

## 7. Technical Architecture

### 7.1 System Components

**Development Platform:** Lovable (AI-powered React development)

#### Frontend
- **Technology:** React.js with shadcn/ui component library
- **Platform:** Lovable (AI-powered development platform)
- **UI Components:** shadcn/ui (Radix UI primitives + Tailwind CSS)
  - Button, Input, Card, Dialog components
  - Progress bar, Checkbox, Radio Group
  - Toast notifications for user feedback
- **Responsibilities:**
  - User interface rendering with shadcn/ui components
  - Form validation
  - CSV parsing (client-side using PapaParse or similar)
  - State management for batch jobs (React hooks/context)
  - Progress tracking
  - Image preview gallery

#### Backend
- **Technology:** Supabase (Backend-as-a-Service)
  - **Database:** PostgreSQL (Supabase managed)
  - **Authentication:** Supabase Auth
  - **Storage:** Supabase Storage for images
  - **Edge Functions:** Supabase Edge Functions (Deno runtime) for API orchestration
- **Responsibilities:**
  - API key management (encrypted storage in PostgreSQL)
  - OpenAI API communication (via Edge Functions)
  - URL fetching and JPEG conversion (server-side)
  - ZIP file generation (server-side via Edge Function)
  - Image persistence (Supabase Storage)
  - Job queue management
  - Real-time updates (Supabase Realtime)

#### Storage
- **Image Storage:** Supabase Storage (object storage)
  - Organized by user ID and batch ID
  - Public or private buckets based on user preferences
  - CDN-enabled for fast image delivery
- **Database:** Supabase PostgreSQL
  - **Tables:**
    - `users` - User authentication and profile data
    - `batches` - Batch job metadata (user_id, created_at, status, parameters)
    - `images` - Image metadata (batch_id, prompt, storage_url, status, error_message)
    - `api_keys` - Encrypted OpenAI API keys (user_id, encrypted_key)
  - **Real-time subscriptions:** For live progress updates during batch generation

---

### 7.2 API Integration Flow

```
User Input (React) → Client-side Validation → Supabase Edge Function
                                                      ↓
                                              Queue Manager (PostgreSQL)
                                                      ↓
                                              Sequential API Calls to OpenAI DALL-E 3
                                                      ↓
                                              Response Handler (receives URL)
                                                      ↓
                                              Fetch Image from OpenAI URL
                                                      ↓
                                              Convert to JPEG (server-side)
                                                      ↓
                                              Supabase Storage (save JPEG)
                                                      ↓
                                              Database Update (PostgreSQL)
                                                      ↓
                                              Frontend Update (Supabase Realtime)
```

**Key Components:**
- **React Frontend:** Handles UI, prompt input, and initiates batch jobs
- **Supabase Edge Functions:** Serverless functions that:
  - Orchestrate OpenAI DALL-E 3 API calls
  - Fetch images from temporary OpenAI URLs
  - Convert images to JPEG format
  - Upload JPEGs to Supabase Storage
- **PostgreSQL Queue:** Tracks batch jobs and individual prompt status
- **Supabase Storage:** Stores permanent JPEG images with public URLs
- **Supabase Realtime:** Pushes progress updates to frontend without polling

**Critical Path:**
1. API call returns temporary URL (60-minute expiry)
2. Edge Function immediately fetches binary image
3. Server-side JPEG conversion (quality: 80-95%)
4. Upload to permanent Supabase Storage
5. Update database with storage URL and metadata
6. Push update to frontend via Realtime

---

### 7.3 Lovable Development Considerations

**Platform Overview:**
Lovable is an AI-powered development platform optimized for React applications with Supabase backend integration.

**Key Advantages:**
1. **Rapid Prototyping:** AI-assisted component generation and iteration
2. **Built-in Integration:** Seamless Supabase connection and configuration
3. **Component Library:** Pre-configured shadcn/ui components reduce development time
4. **TypeScript Support:** Type-safe development out of the box

**Development Phasing Strategy:**
This project will follow a **frontend-first development approach**:

**Phase 1: Frontend Development**
- Build all UI components and pages using React + shadcn/ui
- Implement client-side validation and state management
- Create mock data and simulated flows for testing UX
- Establish user interface patterns and interactions
- Complete visual design and responsiveness
- User acceptance testing on UI/UX without backend

**Phase 2: Backend Development**
- Design and implement database schema in Supabase PostgreSQL
- Configure Row-Level Security (RLS) policies
- Build Supabase Edge Functions for:
  - OpenAI DALL-E 3 API integration
  - URL fetching and JPEG conversion
  - Image storage management
- Set up Supabase Realtime for progress updates
- Implement authentication and API key encryption

**Phase 3: Integration**
- Connect frontend to backend services
- End-to-end testing of complete workflows
- Performance optimization
- Bug fixes and refinements

**Rationale for Frontend-First:**
- Allows early user feedback on interface and experience
- Clarifies backend requirements through UI implementation
- Enables parallel work if team expands
- Reduces rework by validating UX before backend investment

**Development Workflow:**
1. Use Lovable's AI to generate React components for UI pages
2. Configure Supabase project and link to Lovable application
3. Define database schema in Supabase (tables, RLS policies)
4. Create Supabase Edge Functions for OpenAI API integration
5. Implement Supabase Realtime subscriptions for progress updates
6. Deploy directly from Lovable to hosting (Vercel, Netlify, etc.)

**Recommended Libraries:**
- **CSV Parsing:** ~~papaparse~~ (OUT OF SCOPE - CSV upload removed)
- **File Downloads:** `jszip` (client-side ZIP creation) or Edge Function for server-side
- **Image Processing:** `sharp` (server-side JPEG conversion in Edge Functions)
- **State Management:** React Context API or Zustand
- **Form Handling:** React Hook Form with Zod validation
- **HTTP Client:** Supabase client library (built-in)

**Supabase Configuration Needed:**
- Enable Supabase Auth (email/password)
- Create storage bucket for images with RLS policies
- Set up Edge Functions for:
  - OpenAI DALL-E 3 API calls
  - Image fetching from OpenAI URLs
  - JPEG conversion (using sharp or similar)
  - Image upload to Supabase Storage
- Configure Realtime for batch progress updates
- Enable pgcrypto extension for API key encryption

---

### 7.4 Security Requirements

1. **API Key Protection**
   - Encrypt OpenAI API keys using Supabase Vault or pgcrypto
   - Store keys in PostgreSQL with row-level security (RLS)
   - Never expose keys in frontend code or client-side storage
   - API keys only accessible via Supabase Edge Functions
   - Allow users to update/rotate keys through secure endpoints

2. **Data Privacy**
   - User images are private by default using Supabase Storage RLS policies
   - Implement Supabase Authentication (email/password or OAuth)
   - Row-level security policies isolate user data by auth.uid()
   - Secure storage buckets with user-specific access policies

3. **Input Sanitization**
   - Validate all user inputs on both client and server side
   - Prevent injection attacks using parameterized queries
   - Limit file upload sizes (CSV max 5MB via Supabase Storage policies)
   - Rate limiting on Edge Functions to prevent abuse

---

### 7.5 Performance Requirements

1. **Response Time**
   - UI interactions: <200ms
   - Batch job initiation: <1s
   - Individual image generation: 5-15s (dependent on OpenAI API)

2. **Throughput**
   - Support concurrent batch jobs from multiple users
   - Queue system to prevent API rate limit violations

3. **Scalability**
   - Handle 100+ concurrent users
   - Storage capacity for 10,000+ images per user

---

## 8. User Interface Specifications

### 8.1 Page Structure

#### 8.1.1 Prompt Input Page
**Components:**
- Header: Application title and navigation
- Text Area (100 lines max) with line numbers
- Real-time Prompt Counter: "X/100 prompts"
- Parameter Selection Panel:
  - **Style** (radio buttons): Vivid (default) | Natural
  - **Image Size** (radio buttons): (9:16) | (1:1) default
- Visual error indicators (red underline for multi-prompt lines)
- "Generate Images" Button (primary CTA)

**Removed Components (Out of Scope):**
- ~~CSV Upload Button~~
- ~~CSV Template Download Link~~
- ~~Input Method Toggle~~

---

#### 8.1.2 Generation Progress Page
**Components:**
- Progress Bar (visual + percentage)
- Status Text: "Processing prompt 35 of 100"
- Current Prompt Display
- Control Buttons: Pause/Resume, Cancel
- Error Counter: "2 prompts failed"
- Estimated Time Remaining

---

#### 8.1.3 Results Page
**Components:**
- Header: "Generation Complete: X images generated"
- View Toggle: Grid | List
- Image Gallery:
  - Thumbnail preview (JPEG)
  - Associated prompt text (original + revised)
  - Individual download button (.jpg)
  - Checkbox for selection
- Bulk Actions Panel:
  - "Select All" checkbox
  - "Download Selected (ZIP)" button
  - "Delete Selected" button
- Error Summary (if applicable):
  - List of failed prompts with reasons
  - Retry option for failed prompts

**Note:** All images are JPEG format only

---

### 8.2 Responsive Design
- Mobile-optimized layout
- Touch-friendly controls
- Gallery adjusts to screen size
- Minimum supported resolution: 320px width

---

## 9. Non-Functional Requirements

### 9.1 Reliability
- 99.5% uptime (excluding OpenAI API downtime)
- Graceful degradation during API outages
- Data persistence during unexpected disconnections

### 9.2 Maintainability
- Modular code architecture
- Comprehensive error logging
- API version compatibility monitoring

### 9.3 Usability
- Intuitive UI requiring minimal instruction
- Inline help text and tooltips
- Error messages are actionable and user-friendly

### 9.4 Compliance
- Adhere to OpenAI's usage policies
- GDPR compliance for EU users (if applicable)
- Secure handling of user credentials

---

## 10. Future Enhancements (Out of Scope for v1.0)

### 10.1 Planned Features
1. **CSV Upload Support:** Bulk prompt upload via CSV files (currently out of scope)
2. **Additional Model Support:** DALL-E 2, future OpenAI image models
3. **PNG Format Support:** Optional PNG output alongside JPEG
4. **Additional Image Sizes:** Support for all DALL-E 3 sizes (including 1792x1024)
5. **Advanced Parameters:** Quality settings, additional style modifiers
6. **Prompt Library:** Save and reuse common prompts
7. **Batch Scheduling:** Schedule generation jobs for later execution
8. **API Usage Analytics:** Track costs and usage statistics per batch
9. **Image Editing:** Basic post-generation editing tools
10. **Collaboration:** Share batches with team members
11. **Webhook Support:** Notifications when batches complete
12. **Prompt Templates:** Pre-built prompt structures for common use cases
13. **Revised Prompt Comparison:** Side-by-side view of original vs. OpenAI revised prompts
14. **Batch History:** View and re-download previous generation batches

### 10.2 Integration Opportunities
- Figma plugin
- Adobe Creative Suite integration
- Cloud storage sync (Dropbox, Google Drive)
- Social media direct posting

---

## 11. Testing Requirements

### 11.1 Unit Testing
- ~~CSV parsing logic~~ (OUT OF SCOPE)
- URL fetching from OpenAI responses
- JPEG conversion logic
- Input validation functions (including multi-prompt detection)
- API request formatting for DALL-E 3

### 11.2 Integration Testing
- End-to-end batch generation flow with DALL-E 3
- OpenAI API communication
- URL-to-JPEG conversion pipeline
- Supabase Storage upload and retrieval
- ZIP file creation and download
- Database persistence operations
- Realtime progress updates

### 11.3 User Acceptance Testing
- User story validation (text area input flow)
- Edge case scenarios (especially URL expiry handling)
- Cross-browser compatibility
- Mobile responsiveness
- JPEG quality verification

### 11.4 Performance Testing
- Load testing with 100 concurrent users
- Batch processing with maximum prompts (100)
- JPEG conversion speed and quality
- Storage capacity testing
- Memory leak detection during image processing

---

## 12. Release Criteria

### 12.1 MVP Launch Requirements
- [ ] All P0 features implemented and tested
- [ ] Error handling for critical edge cases functional
- [ ] Security review completed
- [ ] User documentation prepared
- [ ] API key setup guide published
- [ ] Performance benchmarks met

### 12.2 Definition of Done
- Feature is code-complete
- Unit tests pass with >80% coverage
- Integration tests pass
- User acceptance criteria validated
- Documentation updated
- No critical or high-priority bugs

---

## 13. Documentation and Support

### 13.1 User Documentation
1. **Getting Started Guide**
   - OpenAI account setup and funding
   - API key generation and configuration
   - First batch generation tutorial with DALL-E 3

2. **Feature Documentation**
   - ~~CSV template usage~~ (OUT OF SCOPE)
   - Text area prompt input best practices
   - Style and size parameter selection guide
   - Batch management best practices
   - JPEG download and ZIP archive usage
   - Troubleshooting common errors

3. **FAQ**
   - API key issues
   - Billing and costs (DALL-E 3 specific)
   - Content policy violations
   - Why JPEG only? (file size, compatibility, quality)
   - Image URL expiry (60-minute window)
   - Prompt revision by OpenAI

### 13.2 Technical Documentation
- API integration guide
- Deployment instructions
- Database schema
- Configuration reference

---

## 14. Open Questions and Decisions Required

1. **Authentication:** How will users authenticate? (Email/password, OAuth, API key only?)
2. **Pricing Model:** Free tier limits? Subscription tiers?
3. **Storage Limits:** How long are images retained? Storage quotas per user?
4. **Multi-tenancy:** Support for team accounts or organization-level access?
5. **Monitoring:** What metrics should be tracked for product analytics?
6. **Deployment:** On-premise option or cloud-only?

---

## 15. Appendix

### 15.1 Glossary
- **Batch Job:** A collection of image generation prompts submitted together
- **DALL-E 3:** OpenAI's latest image generation model with improved quality and prompt adherence
- **Revised Prompt:** OpenAI's enhanced version of the user's original prompt (for safety and clarity)
- **Temporary URL:** OpenAI-provided image URL that expires after 60 minutes
- **URL-to-JPEG Conversion:** Server-side process of fetching image from temporary URL and converting to permanent JPEG
- **Content Policy Violation:** Prompt that violates OpenAI's usage guidelines
- **Sequential Processing:** Executing API calls one after another, not in parallel
- **Edge Function:** Serverless function running on Supabase infrastructure (Deno runtime)

### 15.2 References
- [OpenAI DALL-E 3 API Documentation](https://platform.openai.com/docs/api-reference/images/create)
- [OpenAI Usage Policies](https://openai.com/policies/usage-policies)
- [DALL-E 3 Model Overview](https://platform.openai.com/docs/guides/images)
- [Competitor Analysis: DALL-E Bulk Generator](https://dall-ebulkimagegenerator.com/)
- [Supabase Storage Documentation](https://supabase.com/docs/guides/storage)
- [Supabase Edge Functions Documentation](https://supabase.com/docs/guides/functions)

### 15.3 Revision History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-11 | [Name] | Initial PRD creation |
| 2.0 | 2025-11-12 | [Name] | Major update: Changed to DALL-E 3 model, removed CSV upload, JPEG-only output, URL-to-JPEG conversion, updated parameters (style/size) |

---

## 16. Approval and Sign-off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Manager | | | |
| Engineering Lead | | | |
| Design Lead | | | |
| Stakeholder | | | |

---

**End of Document**
