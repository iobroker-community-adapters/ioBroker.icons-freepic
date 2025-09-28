# ioBroker Adapter Development with GitHub Copilot

**Version:** 0.4.0
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on ioBroker adapter development.

## Project Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

## Adapter-Specific Context
- **Adapter Name**: iobroker.icons-freepic
- **Primary Function**: Provides icon resources from freepic for ioBroker visualizations (webui, vis, etc.)
- **Adapter Type**: visualization-icons (WWW-only, no runtime instance)
- **Configuration**: None required (noConfig: true)
- **Mode**: none (static file serving)
- **Key Features**:
  - Serves static icon files from www/ directory
  - Extracted from ioBroker.habpanel for standalone use
  - Compatible with all ioBroker visualization systems
  - No backend logic or state management required

## Development Guidelines

### File Structure
This is a WWW-only adapter serving static content:
```
www/                    # Static icon files served to frontend
admin/                  # Admin interface files (icons-freepic.png, words.js)
io-package.json         # Adapter metadata and configuration
package.json           # Node.js package configuration
README.md              # Documentation
```

### Icon Management
- All icons are stored in the `www/` directory
- Icons should be optimized for web use (appropriate file sizes)
- Consider copyright and trademark compliance (see README disclaimer)
- Maintain consistent naming conventions for easy discovery
- Icons are served directly to visualization frontends

### Testing

#### Unit Testing
- Use Jest as the primary testing framework for ioBroker adapters
- For WWW-only adapters, focus on:
  - Package file validation (io-package.json, package.json)
  - File structure integrity
  - Icon file accessibility and formats
- Example test structure:
  ```javascript
  describe('icons-freepic adapter', () => {
    test('should have valid package files', () => {
      // Test package.json and io-package.json validity
    });
    
    test('should serve icons from www directory', () => {
      // Test icon file accessibility
    });
  });
  ```

#### Integration Testing

**IMPORTANT**: Use the official `@iobroker/testing` framework for all integration tests. This is the ONLY correct way to test ioBroker adapters.

**Official Documentation**: https://github.com/ioBroker/testing

#### Framework Structure
Integration tests MUST follow this exact pattern:

```javascript
const path = require('path');
const { tests } = require('@iobroker/testing');

// Use tests.integration() with defineAdditionalTests
tests.integration(path.join(__dirname, '..'), {
    defineAdditionalTests({ suite }) {
        suite('Test WWW-only adapter functionality', (getHarness) => {
            let harness;

            before(() => {
                harness = getHarness();
            });

            it('should handle WWW-only adapter setup', function () {
                return new Promise(async (resolve, reject) => {
                    try {
                        harness = getHarness();
                        
                        // Get adapter object
                        const obj = await new Promise((res, rej) => {
                            harness.objects.getObject('system.adapter.icons-freepic.0', (err, o) => {
                                if (err) return rej(err);
                                res(o);
                            });
                        });
                        
                        if (!obj) {
                            return reject(new Error('Adapter object not found'));
                        }

                        console.log('✅ WWW-only adapter object validated');
                        
                        // For WWW-only adapters, no runtime instance starts
                        // Test focuses on object configuration and file serving
                        
                        resolve();
                        
                    } catch (error) {
                        reject(error);
                    }
                });
            }).timeout(30000);
        });
    }
});
```

**Critical Testing Notes for WWW-Only Adapters:**
- No runtime instance to start/stop (mode: "none")
- Focus on static file serving and object configuration
- Test icon file accessibility through HTTP requests
- Validate io-package.json configuration (onlyWWW: true, noConfig: true)
- Use shorter timeouts since no complex initialization occurs

### WWW-Only Adapter Patterns

#### Static File Serving
```javascript
// WWW-only adapters serve files from www/ directory automatically
// No custom server code needed - ioBroker handles HTTP serving

// Example icon URL structure:
// http://iobroker-ip:8082/adapter/icons-freepic/icon-name.png
```

#### Admin Integration
```javascript
// Admin interface for WWW-only adapters (admin/index_m.html if needed)
// Most WWW-only adapters need no admin interface (noConfig: true)

// words.js for translations (if admin interface exists)
systemDictionary = {
    "myColor": {
        "en": "myColor",
        "de": "meineColor",
        // ... other languages
    }
};
```

### Error Handling for Static Content
- Validate icon file formats during development
- Check file permissions and accessibility
- Handle missing icon scenarios gracefully in frontends
- Log warnings for broken icon references

### Code Style and Standards

- Follow JavaScript/TypeScript best practices
- Use semantic versioning for adapter releases
- Include proper JSDoc comments for any utility functions
- Maintain clean directory structure in www/
- Follow ioBroker naming conventions for files and resources

### Copyright and Legal Compliance

**CRITICAL**: This adapter serves icons from freepic. Always ensure:
- Proper attribution as required by freepic license
- Copyright notices in README and relevant files
- Disclaimer about trademark and copyright responsibility
- Regular review of license compliance for all included icons

## CI/CD and Testing Integration

### GitHub Actions for WWW-Only Adapters
WWW-only adapters have simpler CI/CD requirements:

```yaml
# Simplified testing for static content adapters
test:
  runs-on: ubuntu-22.04
  
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Use Node.js 20.x
      uses: actions/setup-node@v4
      with:
        node-version: 20.x
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run package validation tests
      run: npm test
      
    - name: Validate WWW directory structure
      run: |
        if [ ! -d "www" ]; then
          echo "ERROR: www directory not found for WWW-only adapter"
          exit 1
        fi
        echo "✅ WWW directory structure validated"
```

### Package.json Script Integration
Minimal scripts for WWW-only adapters:
```json
{
  "scripts": {
    "test:package": "mocha test/package --exit",
    "test": "npm run test:package"
  }
}
```

### Best Practices for Icon Adapters
- Regular icon optimization to reduce file sizes
- Automated checks for broken icon references
- Version control for icon updates and additions
- Documentation of icon sources and licenses
- Validation of icon accessibility from web interfaces

## Legal and Licensing

### Icon Attribution Requirements
- Maintain attribution for all freepic icons as required
- Include proper disclaimers about trademark usage
- Regular audits of icon licensing compliance
- Clear documentation of icon sources

### User Responsibility
- Users must verify copyright and trademark compliance
- Adapter provides icons but users are responsible for proper usage
- Include clear disclaimers in documentation and README

This adapter focuses on providing a curated collection of icons for ioBroker visualizations while maintaining legal compliance and ease of use.