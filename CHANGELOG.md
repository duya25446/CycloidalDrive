# Changelog

All notable changes to the Cycloidal Drive Generator script will be documented in this file.

## [2.0.0] - 2026-02-08

### Fixed
- **Python Syntax Error**: Fixed namedtuple definition on line 587 - added missing comma between field names
- **Function Name Typo**: Renamed `settingComandInputsItem` to `settingCommandInputsItem` (added missing 'm')
- **Function Call**: Updated function call on line 720 to match corrected function name

### Changed
- **Pickle Protocol**: Added `protocol=pickle.HIGHEST_PROTOCOL` parameter to `pickle.dumps()` for better compatibility
- **Error Handling**: Added exception handling to `pickle.loads()` to gracefully handle unpickling errors (UnpicklingError, EOFError, AttributeError)
- **String Quotes**: Normalized string quotes in namedtuple definition from single to double quotes for consistency

### Updated
- **Manifest Version**: Bumped version from 1.0.0 to 2.0.0
- **Manifest Description**: Updated description to indicate compatibility with modern Fusion360
- **README**: Added "What's New in v2.0.0" section documenting all changes

### Technical Notes
- All mathematical algorithms (Simpson integration, Newton's method, cycloidal curve calculations) remain unchanged
- UI layout and interaction logic preserved
- Code architecture and variable naming conventions maintained (except for the function name typo fix)

## [1.0.0] - 2019 (Original Release)

### Features
- Initial release of cycloidal drive generator for Fusion360
- Support for creating:
  - Cycloidal gear with trochoidal parallel curve
  - Ring pins configuration
  - Center hole
  - Around holes
  - Output disk pins
- Interactive parameter input dialog
- Automatic sketch generation with geometric constraints
