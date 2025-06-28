# Bulk Move Plugin - Implementation Plan

**Version**: 2.0
**Target Release**: Q3 2025
**Document Version**: 2024-06-27
**Estimated Effort**: 10 weeks (2 developers)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Project Architecture](#project-architecture)
3. [Technology Stack](#technology-stack)
4. [Development Phases](#development-phases)
5. [Implementation Details](#implementation-details)
6. [Testing Strategy](#testing-strategy)
<!-- 7. [Performance Requirements](#performance-requirements)
8. [Risk Management](#risk-management)
9. [Quality Assurance](#quality-assurance)
10. [Deployment Strategy](#deployment-strategy)
11. [Maintenance Plan](#maintenance-plan) -->

---

## Executive Summary

### Project Goals
Develop a production-ready Obsidian plugin for bulk file operations with enterprise-grade safety, performance, and accessibility. The plugin must handle 50K+ files without UI degradation while maintaining zero data loss guarantee.

### Key Success Criteria
- **Reliability**: 99.9% operation success rate with complete rollback capability
- **Performance**: Handle 50K files with <2s load time and 60fps UI
- **Safety**: Two-phase commit system with atomic undo within 5 minutes
- **Accessibility**: WCAG 2.1 AA compliance with full keyboard navigation
- **Adoption**: 10K+ active users within 6 months

### Resource Requirements
- **Development Team**: 2 senior developers (TypeScript, Svelte, Obsidian API)
- **Timeline**: 10 weeks development + 2 weeks buffer
- **Testing Environment**: Windows, macOS, Linux, iOS, Android devices
- **External Dependencies**: Obsidian API, modern web browsers

---

## Project Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Obsidian Plugin Host                   │
├─────────────────────────────────────────────────────────────┤
│  UI Layer (Svelte)                                        │
│  ├── BulkMoveView.ts        ├── Modals/               │
│  ├── FileList.svelte       ├── Components/           │
│  └── Mobile/               └── Accessibility/        │
├─────────────────────────────────────────────────────────────┤
│  Application Logic                                         │
│  ├── Store Management      ├── Operation Engine      │
│  ├── Search Integration    ├── Conflict Resolution   │
│  └── Progress Tracking     └── Template System       │
├─────────────────────────────────────────────────────────────┤
│  Core Services                                             │
│  ├── File System Manager   ├── Backup Manager        │
│  ├── API Compatibility     ├── Performance Monitor   │
│  └── Logger                └── Validation Service    │
├─────────────────────────────────────────────────────────────┤
│  Obsidian API Abstraction Layer                           │
│  ├── File Operations       ├── Search Interface      │
│  ├── Workspace Events      ├── Settings Manager      │
│  └── Mobile Detection      └── Plugin Integration    │
└─────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
src/
├── main.ts                          # Plugin entry point
├── manifest.json                    # Plugin manifest
├── api/                            # Obsidian API abstractions
│   ├── ObsidianApiAdapter.ts       # API compatibility layer
│   ├── FileSystemManager.ts        # File operation wrapper
│   ├── SearchManager.ts            # Search abstraction
│   └── WorkspaceManager.ts         # Workspace event handling
├── core/                           # Business logic
│   ├── BulkMoveEngine.ts          # Two-phase commit engine
│   ├── ConflictResolver.ts        # Conflict detection/resolution
│   ├── BackupManager.ts           # Backup and restore system
│   ├── ProgressTracker.ts         # Operation monitoring
│   ├── TemplateEngine.ts          # File naming templates
│   └── OperationJournal.ts        # Operation logging
├── store/                          # State management
│   ├── BulkMoveStore.ts           # Main application state
│   ├── SelectionManager.ts        # File selection state
│   ├── SettingsStore.ts           # Plugin configuration
│   └── types.ts                   # State type definitions
├── ui/                            # User interface
│   ├── views/
│   │   ├── BulkMoveView.ts        # Main panel view
│   │   ├── MobileView.ts          # Mobile-optimized UI
│   │   └── SettingsTab.ts         # Plugin settings
│   ├── components/
│   │   ├── FileList.svelte        # Virtual file list
│   │   ├── SearchBar.svelte       # Search input with history
│   │   ├── ActionPanel.svelte     # Operations toolbar
│   │   ├── ProgressBar.svelte     # Progress indicator
│   │   └── SelectionControls.svelte# Selection tools
│   ├── modals/
│   │   ├── DryRunModal.ts         # Operation preview
│   │   ├── FolderPickerModal.ts   # Folder selection
│   │   ├── ConflictModal.ts       # Conflict resolution
│   │   └── ProgressModal.ts       # Progress dialog
│   └── styles/
│       ├── main.css               # Core styles
│       ├── mobile.css             # Mobile-specific styles
│       └── accessibility.css      # A11y enhancements
├── utils/                         # Utility functions
│   ├── Logger.ts                  # Structured logging
│   ├── Performance.ts             # Performance monitoring
│   ├── Validation.ts              # Input validation
│   ├── Platform.ts                # Platform detection
│   └── Debounce.ts               # Utility functions
├── types/                         # TypeScript definitions
│   ├── api.ts                     # API type definitions
│   ├── operations.ts              # Operation type definitions
│   ├── ui.ts                      # UI component types
│   └── global.ts                  # Global type declarations
└── locales/                       # Internationalization
    ├── en.json                    # English translations
    ├── ja.json                    # Japanese translations
    └── index.ts                   # i18n loader
```

---

## Technology Stack

### Core Technologies

| Component | Technology | Version | Justification |
|-----------|------------|---------|---------------|
| **Framework** | TypeScript | 5.1+ | Type safety, modern features, Obsidian compatibility |
| **UI Library** | Svelte | 4.0+ | Lightweight, reactive, excellent performance |
| **State Management** | Zustand + Immer | Latest | Simple, performant, immutable updates |
| **Virtual Scrolling** | TanStack Virtual | 3.0+ | High-performance list rendering |
| **Async Control** | p-queue | 7.0+ | Concurrency control and rate limiting |
| **Template Engine** | Handlebars | 4.7+ | Safe, powerful template processing |
| **Build Tool** | esbuild | 0.18+ | Fast builds, ES6+ support |

### Development Tools

| Category | Tool | Purpose |
|----------|------|---------|
| **Testing** | Vitest + Playwright | Unit and E2E testing |
| **Linting** | ESLint + Prettier | Code quality and formatting |
| **Type Checking** | TypeScript compiler | Static type validation |
| **Bundling** | esbuild | Plugin compilation |
| **CI/CD** | GitHub Actions | Automated testing and deployment |
| **Documentation** | TypeDoc | API documentation generation |

### External Dependencies

```json
{
  "dependencies": {
    "zustand": "^4.4.0",
    "immer": "^10.0.0",
    "@tanstack/virtual-core": "^3.0.0",
    "p-queue": "^7.4.0",
    "handlebars": "^4.7.0",
    "fuzzysort": "^2.0.0"
  },
  "devDependencies": {
    "typescript": "^5.1.0",
    "esbuild": "^0.18.0",
    "vitest": "^0.34.0",
    "playwright": "^1.37.0",
    "eslint": "^8.47.0",
    "prettier": "^3.0.0",
    "@types/node": "^20.5.0"
  }
}
```

---

## Development Phases

### Phase 1: Foundation (Weeks 1-2)
**Goal**: Establish core architecture and basic functionality

#### Week 1: Project Setup & API Layer
- [ ] **Repository Setup**
  - Initialize TypeScript project with proper tooling
  - Configure esbuild for Obsidian plugin compilation
  - Set up GitHub Actions for CI/CD
  - Create development environment documentation

- [ ] **API Compatibility Layer**
  - Implement `ObsidianApiAdapter` with feature detection
  - Create fallback mechanisms for API changes
  - Develop `FileSystemManager` with atomic operations
  - Build `SearchManager` with multiple search strategies

- [ ] **Basic Plugin Structure**
  - Create main plugin entry point and lifecycle
  - Implement basic settings management
  - Set up logging system with configurable levels
  - Create plugin manifest and metadata

#### Week 2: Core Architecture & Store
- [ ] **State Management**
  - Implement `BulkMoveStore` with Zustand + Immer
  - Create `SelectionManager` for efficient file selection
  - Develop store persistence and hydration
  - Build reactive update mechanisms

- [ ] **UI Foundation**
  - Create `BulkMoveView` as Obsidian ItemView
  - Implement basic Svelte component structure
  - Set up virtual scrolling with TanStack Virtual
  - Develop responsive layout system

- [ ] **Search Integration**
  - Build search synchronization with Obsidian
  - Implement search history management
  - Create live filtering capabilities
  - Add query validation and error handling

**Deliverables**: Working plugin skeleton with basic UI and search integration

### Phase 2: Core Engine (Weeks 3-4)
**Goal**: Implement the heart of bulk operations with safety guarantees

#### Week 3: Two-Phase Commit Engine
- [ ] **Operation Engine**
  - Implement `BulkMoveEngine` with two-phase commit
  - Create operation planning and validation
  - Build atomic execution with rollback capability
  - Develop progress tracking and cancellation

- [ ] **Conflict Resolution**
  - Implement `ConflictResolver` with multiple strategies
  - Create conflict detection algorithms
  - Build interactive conflict resolution UI
  - Add batch conflict handling

- [ ] **Backup System**
  - Develop `BackupManager` for pre-operation snapshots
  - Implement incremental backup strategies
  - Create backup verification and integrity checks
  - Build restore functionality with selective recovery

#### Week 4: Operation Features
- [ ] **File Operations**
  - Implement move, copy, rename, delete operations
  - Add concurrency control with platform optimization
  - Create retry mechanisms with exponential backoff
  - Build operation journaling for audit trails

- [ ] **Undo System**
  - Develop atomic undo with complete operation reversal
  - Implement time-based undo expiration
  - Create undo UI with operation details
  - Add partial undo for failed operations

- [ ] **Template System**
  - Implement Handlebars-based naming templates
  - Create template validation and preview
  - Add built-in template library
  - Build custom variable support

**Deliverables**: Fully functional bulk operations with safety guarantees

### Phase 3: User Experience (Weeks 5-6)
**Goal**: Polish UI/UX and implement mobile experience

#### Week 5: Desktop UI & Accessibility
- [ ] **Enhanced UI Components**
  - Complete `FileList.svelte` with virtual scrolling
  - Implement `ActionPanel.svelte` with contextual actions
  - Create `SearchBar.svelte` with advanced features
  - Build `ProgressModal.ts` for operation tracking

- [ ] **Accessibility Implementation**
  - Add comprehensive ARIA labels and descriptions
  - Implement full keyboard navigation
  - Create screen reader optimized interactions
  - Add high contrast and reduced motion support

- [ ] **Modal System**
  - Implement `DryRunModal.ts` for operation preview
  - Create `FolderPickerModal.ts` with fuzzy search
  - Build `ConflictModal.ts` for resolution workflow
  - Add modal accessibility and focus management

#### Week 6: Mobile Experience
- [ ] **Mobile UI Optimization**
  - Create `MobileView.ts` with touch-optimized layout
  - Implement gesture-based interactions
  - Add responsive design with breakpoints
  - Build mobile-specific components

- [ ] **Touch Interactions**
  - Implement long-press selection mode
  - Create swipe gestures for navigation
  - Add haptic feedback support
  - Build one-handed operation support

- [ ] **Performance Optimization**
  - Optimize for mobile hardware constraints
  - Implement intelligent caching strategies
  - Add memory management for mobile
  - Create performance monitoring dashboard

**Deliverables**: Polished UI with full accessibility and mobile support

### Phase 4: Performance & Testing (Weeks 7-8)
**Goal**: Optimize performance and implement comprehensive testing

#### Week 7: Performance Optimization
- [ ] **Performance Engineering**
  - Implement adaptive concurrency based on system resources
  - Create memory-efficient virtual scrolling
  - Add intelligent caching and lazy loading
  - Build performance monitoring and profiling

- [ ] **Scalability Testing**
  - Test with 50K+ files for performance validation
  - Optimize memory usage and garbage collection
  - Implement background processing with Web Workers
  - Create performance regression testing

- [ ] **Error Handling & Recovery**
  - Implement comprehensive error handling strategies
  - Create automatic error recovery mechanisms
  - Add detailed error logging and reporting
  - Build user-friendly error messages and guidance

#### Week 8: Testing Implementation
- [ ] **Unit Testing**
  - Create comprehensive test suite with Vitest
  - Test all core business logic and utilities
  - Implement mock Obsidian API for testing
  - Add test coverage reporting and enforcement

- [ ] **Integration Testing**
  - Build E2E test suite with Playwright
  - Test complete user workflows
  - Create cross-platform compatibility tests
  - Add performance benchmark tests

- [ ] **User Acceptance Testing**
  - Conduct testing with real-world scenarios
  - Validate accessibility compliance
  - Test mobile device compatibility
  - Gather feedback and iterate on UX

**Deliverables**: Optimized, thoroughly tested plugin ready for release

### Phase 5: Release Preparation (Weeks 9-10)
**Goal**: Finalize documentation, CI/CD, and prepare for community release

#### Week 9: Documentation & Polish
- [ ] **Documentation**
  - Create comprehensive user documentation
  - Write developer API documentation with TypeDoc
  - Build troubleshooting and FAQ guides
  - Create video tutorials and examples

- [ ] **Internationalization**
  - Implement i18n system with locale support
  - Create English and Japanese translations
  - Add locale detection and switching
  - Build translation contribution workflow

- [ ] **Final Polish**
  - Conduct final UI/UX review and refinement
  - Optimize bundle size and loading performance
  - Add final accessibility and compatibility testing
  - Create release notes and changelog

#### Week 10: Release & Deployment
- [ ] **CI/CD Pipeline**
  - Finalize automated testing and deployment
  - Create release automation with GitHub Actions
  - Set up community plugin submission workflow
  - Implement automated version management

- [ ] **Community Release**
  - Submit to Obsidian Community Plugins
  - Create release announcement and marketing
  - Set up community support channels
  - Monitor initial user feedback and issues

- [ ] **Post-Release Support**
  - Implement crash reporting and analytics
  - Create issue triage and response procedures
  - Plan hotfix deployment capabilities
  - Set up long-term maintenance schedule

**Deliverables**: Production-ready plugin released to community

---

## Implementation Details

### Core Service Implementations

#### ObsidianApiAdapter Implementation
```typescript
class ObsidianApiAdapter {
  private version: string;
  private capabilities: Set<string>;
  private fallbackStrategies: Map<string, Function>;

  constructor(private app: App) {
    this.detectCapabilities();
    this.setupFallbacks();
  }

  private detectCapabilities(): void {
    // Feature detection for different Obsidian versions
    this.capabilities.add('basic-file-operations');

    if (this.hasSearchEvents()) {
      this.capabilities.add('search-events');
    }

    if (this.hasModernMetadata()) {
      this.capabilities.add('metadata-api');
    }
  }

  async searchFiles(query: string): Promise<TFile[]> {
    if (this.capabilities.has('search-events')) {
      return this.searchWithEvents(query);
    }

    return this.fallbackStrategies.get('search')(query);
  }
}
```

#### BulkMoveEngine Implementation
```typescript
class BulkMoveEngine {
  async executeBulkOperation(
    operation: BulkOperation,
    options: OperationOptions
  ): Promise<OperationResult> {

    // Phase 1: Validation and Planning
    const plan = await this.createExecutionPlan(operation);
    const conflicts = await this.detectConflicts(plan);

    if (conflicts.length > 0 && !options.autoResolve) {
      const resolution = await this.resolveConflicts(conflicts);
      plan.applyResolution(resolution);
    }

    // Phase 2: Backup Creation
    const backup = await this.backupManager.createSnapshot(
      plan.affectedFiles,
      options.backupStrategy
    );

    // Phase 3: Atomic Execution
    const journal = new OperationJournal();
    try {
      const result = await this.executeWithJournal(plan, journal, options);

      // Phase 4: Verification
      const verification = await this.verifyExecution(result);
      if (!verification.success) {
        await this.rollbackFromJournal(journal);
        throw new OperationError('Verification failed', verification.errors);
      }

      // Phase 5: Cleanup and Finalization
      await this.finalizeOperation(result, backup, journal);
      return result;

    } catch (error) {
      await this.handleOperationFailure(error, journal, backup);
      throw error;
    }
  }

  private async executeWithJournal(
    plan: ExecutionPlan,
    journal: OperationJournal,
    options: OperationOptions
  ): Promise<OperationResult> {

    const queue = new PQueue({
      concurrency: this.calculateOptimalConcurrency(),
      interval: 50,
      intervalCap: 1
    });

    const progressTracker = new ProgressTracker(plan.operations.length);

    const results = await Promise.allSettled(
      plan.operations.map((operation, index) =>
        queue.add(async () => {
          try {
            const result = await this.executeAtomicOperation(operation);
            journal.recordSuccess(operation, result);
            progressTracker.incrementProgress(index, operation.file.path);
            return result;
          } catch (error) {
            journal.recordFailure(operation, error);
            progressTracker.recordError(index, error);
            throw error;
          }
        })
      )
    );

    return this.aggregateResults(results, progressTracker);
  }
}
```

#### Virtual File List Implementation
```typescript
// components/FileList.svelte
<script lang="ts">
  import { createVirtualizer } from '@tanstack/virtual-core';
  import { onMount } from 'svelte';
  import type { TFile } from 'obsidian';

  export let files: TFile[];
  export let selectedPaths: Set<string>;
  export let onSelectionChange: (path: string, selected: boolean) => void;

  let containerElement: HTMLElement;
  let virtualizer: Virtualizer<HTMLElement, HTMLElement>;

  const ITEM_HEIGHT = 32;
  const OVERSCAN = 5;

  onMount(() => {
    virtualizer = createVirtualizer({
      count: files.length,
      getScrollElement: () => containerElement,
      estimateSize: () => ITEM_HEIGHT,
      overscan: OVERSCAN,
    });

    return () => virtualizer?.destroy();
  });

  $: if (virtualizer && files) {
    virtualizer.setOptions({
      count: files.length,
    });
  }

  function handleSelectionChange(file: TFile, event: Event) {
    const checkbox = event.target as HTMLInputElement;
    onSelectionChange(file.path, checkbox.checked);
  }

  function handleKeyDown(event: KeyboardEvent, file: TFile) {
    if (event.key === ' ' || event.key === 'Enter') {
      event.preventDefault();
      const isSelected = selectedPaths.has(file.path);
      onSelectionChange(file.path, !isSelected);
    }
  }
</script>

<div
  bind:this={containerElement}
  class="file-list-container"
  style="height: 400px; overflow: auto;"
  role="listbox"
  aria-label="Files available for bulk operations"
>
  <div style="height: {virtualizer?.getTotalSize() ?? 0}px; position: relative;">
    {#each virtualizer?.getVirtualItems() ?? [] as item (item.key)}
      {@const file = files[item.index]}
      <div
        class="file-item"
        class:selected={selectedPaths.has(file.path)}
        style="
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: {item.size}px;
          transform: translateY({item.start}px);
        "
        role="option"
        aria-selected={selectedPaths.has(file.path)}
        tabindex="0"
        on:keydown={(e) => handleKeyDown(e, file)}
      >
        <label class="file-item-label">
          <input
            type="checkbox"
            checked={selectedPaths.has(file.path)}
            on:change={(e) => handleSelectionChange(file, e)}
            aria-describedby="file-{item.index}-details"
          />
          <span class="file-name">{file.name}</span>
          <span class="file-path" id="file-{item.index}-details">{file.path}</span>
        </label>
      </div>
    {/each}
  </div>
</div>

<style>
  .file-list-container {
    border: 1px solid var(--background-modifier-border);
    border-radius: 4px;
  }

  .file-item {
    display: flex;
    align-items: center;
    padding: 0 8px;
    border-bottom: 1px solid var(--background-modifier-border-hover);
    transition: background-color 0.1s ease;
  }

  .file-item:hover {
    background-color: var(--background-modifier-hover);
  }

  .file-item.selected {
    background-color: var(--background-modifier-active-hover);
  }

  .file-item:focus {
    outline: 2px solid var(--interactive-accent);
    outline-offset: -2px;
  }

  .file-item-label {
    display: flex;
    align-items: center;
    width: 100%;
    cursor: pointer;
    gap: 8px;
  }

  .file-name {
    font-weight: 500;
    color: var(--text-normal);
  }

  .file-path {
    font-size: 0.9em;
    color: var(--text-muted);
    margin-left: auto;
  }

  input[type="checkbox"] {
    margin: 0;
  }

  @media (max-width: 768px) {
    .file-item {
      padding: 12px 16px;
      min-height: 48px;
    }

    .file-path {
      display: none;
    }
  }
</style>
```

---

## Testing Strategy

### Testing Pyramid

```
                    ┌─────────────────┐
                    │   Manual QA     │ ← 5% (Critical user journeys)
                    │   & Acceptance  │
                ┌───┴─────────────────┴───┐
                │     E2E Tests           │ ← 15% (Key workflows)
                │   (Playwright)          │
            ┌───┴─────────────────────────┴───┐
            │     Integration Tests           │ ← 30% (API interactions)
            │    (Vitest + Mocks)             │
        ┌───┴─────────────────────────────────┴───┐
        │         Unit Tests                      │ ← 50% (Business logic)
        │      (Vitest + Jest)                    │
        └─────────────────────────────────────────┘
```

### Unit Testing Strategy

#### Core Business Logic Tests
```typescript
// tests/unit/core/BulkMoveEngine.test.ts
describe('BulkMoveEngine', () => {
  let engine: BulkMoveEngine;
  let mockFileSystem: jest.Mocked<FileSystemManager>;
  let mockBackupManager: jest.Mocked<BackupManager>;

  beforeEach(() => {
    mockFileSystem = createMockFileSystem();
    mockBackupManager = createMockBackupManager();
    engine = new BulkMoveEngine(mockFileSystem, mockBackupManager);
  });

  describe('executeBulkOperation', () => {
    it('should successfully move files without conflicts', async () => {
      const files = createMockFiles(['file1.md', 'file2.md']);
      const operation = createMoveOperation(files, '/target');

      mockFileSystem.checkConflicts.mockResolvedValue([]);
      mockFileSystem.moveFile.mockResolvedValue(true);

      const result = await engine.executeBulkOperation(operation);

      expect(result.success).toBe(true);
      expect(result.movedFiles).toHaveLength(2);
      expect(mockBackupManager.createSnapshot).toHaveBeenCalled();
    });

    it('should handle conflicts with rename strategy', async () => {
      const files = createMockFiles(['existing.md']);
      const operation = createMoveOperation(files, '/target');
      const conflicts = [createConflict('existing.md', 'rename')];

      mockFileSystem.checkConflicts.mockResolvedValue(conflicts);
      mockFileSystem.moveFile.mockResolvedValue(true);

      const result = await engine.executeBulkOperation(operation, {
        conflictResolution: 'rename'
      });

      expect(result.success).toBe(true);
      expect(mockFileSystem.moveFile).toHaveBeenCalledWith(
        expect.anything(),
        expect.stringMatching(/existing-\d+\.md/)
      );
    });

    it('should rollback on operation failure', async () => {
      const files = createMockFiles(['file1.md']);
      const operation = createMoveOperation(files, '/target');

      mockFileSystem.moveFile.mockRejectedValue(new Error('Permission denied'));

      await expect(engine.executeBulkOperation(operation)).rejects.toThrow();
      expect(mockBackupManager.restoreSnapshot).toHaveBeenCalled();
    });
  });
});
```

#### Store Management Tests
```typescript
// tests/unit/store/BulkMoveStore.test.ts
describe('BulkMoveStore', () => {
  let store: BulkMoveStore;

  beforeEach(() => {
    store = createBulkMoveStore();
  });

  describe('file selection', () => {
    it('should select single file', () => {
      const file = createMockFile('test.md');

      store.selectFile(file.path, true);

      expect(store.getState().selection.selectedPaths.has(file.path)).toBe(true);
      expect(store.getState().selection.totalSelected).toBe(1);
    });

    it('should handle range selection', () => {
      const files = createMockFiles(['a.md', 'b.md', 'c.md', 'd.md']);
      store.setSearchResults(files);

      store.selectRange(0, 2);

      expect(store.getState().selection.totalSelected).toBe(3);
      expect(store.getState().selection.selectedPaths.has('a.md')).toBe(true);
      expect(store.getState().selection.selectedPaths.has('c.md')).toBe(true);
    });
  });

  describe('search integration', () => {
    it('should update results when search changes', async () => {
      const mockSearch = jest.fn().mockResolvedValue(createMockFiles(['result.md']));
      store.setSearchFunction(mockSearch);

      await store.updateSearchQuery('test query');

      expect(mockSearch).toHaveBeenCalledWith('test query');
      expect(store.getState().results.files).toHaveLength(1);
      expect(store.getState().searchHistory).toContain('test query');
    });
  });
});
```

### Integration Testing Strategy

#### API Integration Tests
```typescript
// tests/integration/ObsidianApiAdapter.test.ts
describe('ObsidianApiAdapter Integration', () => {
  let adapter: ObsidianApiAdapter;
  let mockApp: MockApp;

  beforeEach(() => {
    mockApp = createMockObsidianApp();
    adapter = new ObsidianApiAdapter(mockApp);
  });

  it('should search files using modern API when available', async () => {
    mockApp.workspace.trigger = jest.fn();
    mockApp.workspace.on = jest.fn().mockImplementation((event, callback) => {
      if (event === 'search:results') {
        setTimeout(() => callback(createMockFiles(['result.md'])), 10);
      }
    });

    const results = await adapter.searchFiles('test');

    expect(mockApp.workspace.trigger).toHaveBeenCalledWith('search:query', 'test');
    expect(results).toHaveLength(1);
  });

  it('should fallback to manual search when modern API unavailable', async () => {
    mockApp.workspace.trigger = undefined;
    mockApp.vault.getMarkdownFiles.mockReturnValue(
      createMockFiles(['test.md', 'other.md'])
    );

    const results = await adapter.searchFiles('test');

    expect(results).toHaveLength(1);
    expect(results[0].name).toBe('test.md');
  });
});
```
