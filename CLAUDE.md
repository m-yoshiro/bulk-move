# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Obsidian plugin for bulk file operations called "Bulk Move". The plugin enables users to efficiently organize large Obsidian vaults by providing powerful batch file operations with enterprise-grade safety and performance.

## Development Commands

- `npm run dev` - Start development build with watch mode
- `npm run build` - Build for production (includes TypeScript checking)
- `npm run version` - Bump version and update manifest files

## Architecture

The project follows a layered architecture:

- **UI Layer**: Svelte components with virtual scrolling for handling large file lists
- **Application Logic**: Store management, operation engine, conflict resolution
- **Core Services**: File system manager, backup manager, performance monitor
- **Obsidian API Layer**: Compatibility layer for different Obsidian versions

### Key Components

- `main.ts` - Plugin entry point extending Obsidian's Plugin class
- `BulkMoveEngine` - Two-phase commit system for atomic operations
- `ConflictResolver` - Handles file naming conflicts during operations
- `BackupManager` - Creates snapshots and enables rollback
- Virtual scrolling file list for performance with 50K+ files

## Core Features

- **Bulk Operations**: Move, copy, rename, delete files in batches
- **Safety System**: Pre-operation backups with atomic undo capability
- **Smart Search**: Integration with Obsidian's search with history
- **Conflict Resolution**: Automatic handling of file naming conflicts
- **Performance**: Optimized for large vaults (50K+ files)
- **Mobile Support**: Touch-optimized UI for mobile devices

## Technology Stack

- TypeScript 5.1+ for type safety
- Svelte 4.0+ for reactive UI components
- esbuild for fast compilation
- Zustand + Immer for state management
- TanStack Virtual for virtual scrolling
- p-queue for concurrency control

## Development Notes

- The plugin uses a two-phase commit system to ensure data safety
- Virtual scrolling is implemented to handle large file lists efficiently
- The API compatibility layer handles different Obsidian versions
- Mobile-first design with touch-optimized interactions
- Comprehensive backup system prevents data loss

## File Structure

Key directories:
- `src/api/` - Obsidian API abstractions
- `src/core/` - Business logic and operation engine
- `src/store/` - State management
- `src/ui/` - Svelte components and views
- `src/utils/` - Utility functions
- `docs/` - Feature specifications and implementation plans