# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

cc-reverse is a Cocos Creator reverse engineering tool that extracts and reconstructs source code and resources from compiled Cocos Creator games. The tool parses compiled JavaScript, extracts assets, and generates a working Cocos Creator project structure.

## Development Commands

```bash
# Start development mode with nodemon
npm run dev

# Run the tool directly
npm start -- --path <source-path>

# Lint code
npm run lint

# Run tests
npm test

# Build project
npm run build

# Global usage after installation
cc-reverse --path <source-path> --output <output-path>

# Force specific Cocos Creator version (when auto-detection fails)
cc-reverse --path <source-path> --version-hint 2.4.x
```

## Architecture Overview

The project follows a modular architecture with clear separation of concerns:

### Core Processing Pipeline
1. **reverseEngine.js** - Main orchestrator that coordinates the entire reverse engineering process
2. **codeAnalyzer.js** - Parses compiled JavaScript using Babel AST, extracts modules and generates TypeScript files
3. **resourceProcessor.js** - Handles asset extraction (textures, audio, animations, scenes)
4. **projectGenerator.js** - Creates Cocos Creator project configuration files
5. **converters.js** - Handles format conversions (JSON to PLIST, sprite atlas processing)

### Key Data Flow
- Input: Compiled Cocos Creator project (supports both 2.3.x and 2.4.x structures)
  - 2.3.x: `res/`, `src/settings.js`, `src/project.js`
  - 2.4.x: `assets/`, `main.js`/`settings.js`, `project.js`/`main.js`
- Processing: Version detection → AST parsing → Resource extraction → Code generation → Project file creation
- Output: Reconstructed Cocos Creator project with TypeScript scripts and organized assets

### Configuration System
- Default configuration in `src/config/configLoader.js`
- User configuration via `cc-reverse.config.js` in project root
- Supports output formatting, code generation preferences, and asset processing options

### Global State Management
The tool uses global variables for cross-module communication:
- `global.config` - Configuration settings
- `global.paths` - File system paths (source, output, temp, etc.)
- `global.settings` - Parsed CCSettings from source project
- `global.verbose` - Debug logging flag
- `global.cocosVersion` - Detected Cocos Creator version ('2.3.x' or '2.4.x')

### Resource Processing Types
- **Scenes** (.fire files) - Game scenes with node hierarchies
- **Scripts** (.ts files) - TypeScript components and logic
- **Audio** - Sound files with metadata
- **Animations** (.anim files) - Animation clips
- **Textures/Sprites** - Image assets and sprite frames
- **Meta files** - Cocos Creator asset metadata (.meta files)

## Key Implementation Details

### UUID Handling
- Uses custom UUID utilities in `src/utils/uuidUtils.js` for Cocos Creator's UUID system
- Decodes compressed UUIDs and generates new ones for reconstructed assets

### AST Processing
- Leverages Babel parser to analyze compiled JavaScript
- Extracts module definitions and reconstructs TypeScript source code
- Processes require/module/exports patterns specific to Cocos Creator builds

### File Management
- Centralized file operations in `src/utils/fileManager.js`
- Handles directory creation, file copying, and cleanup
- Supports both text and binary asset processing