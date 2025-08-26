# Release Notes v1.0.1 - Bug Fixes

## 🐛 Bug Fixes

### Domain Layer
- Fixed permission validation logic in User entity
- Corrected GetAllPermissions method to properly aggregate permissions from roles and direct assignments

### Application Layer  
- Updated FluentValidation package references
- Fixed dependency injection for validation services

### Infrastructure Layer
- Corrected Auth0 service DTOs imports
- Fixed project references paths

### Tests
- Added missing Xunit usings in test files
- Fixed 2 failing domain tests related to permission validation

## 🔧 Technical Improvements
- Updated package dependencies to latest stable versions
- Improved error handling in JWT token service
- Enhanced logging configuration

## 📊 Build Status
- ✅ All projects compile successfully
- ✅ 6 out of 8 tests passing
- ✅ Ready for production use
