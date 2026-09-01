# Agent Instructions for Elliptic Cryptography Library

## Project Overview
**elliptic** is a high-performance JavaScript implementation of elliptic curve cryptography (v6.6.1). It provides ECDSA, EdDSA, and ECDH protocol implementations for cryptographic operations like key generation, signing, and verification.

**Key Resources**: [README.md](README.md), [GitHub](https://github.com/indutny/elliptic)

## Quick Commands

### Testing & Linting
```bash
npm test                    # Run linting + unit tests (recommended)
npm run unit               # Run unit tests only
npm run lint               # Check code style
npm run lint:fix           # Auto-fix code style issues
grunt dist                 # Build browser bundles (dist/elliptic.js, dist/elliptic.min.js)
```

**Test Coverage**: Tests run via Mocha with Istanbul coverage. Entry point: [test/index.js](test/index.js)

## Architecture & Module Structure

### Core Entry Point
[lib/elliptic.js](lib/elliptic.js) - Exports main API:
- `elliptic.ec` - ECDSA protocol
- `elliptic.eddsa` - EdDSA protocol  
- `elliptic.curve` - Low-level curve primitives
- `elliptic.curves` - Preset curves (secp256k1, p256, ed25519, etc.)

### Main Modules

#### Curve Implementations
- [lib/elliptic/curve/base.js](lib/elliptic/curve/base.js) - Base curve class with common operations
- [lib/elliptic/curve/short.js](lib/elliptic/curve/short.js) - Short Weierstrass curves (NIST, secp256k1)
- [lib/elliptic/curve/edwards.js](lib/elliptic/curve/edwards.js) - Edwards curves (Ed25519, etc.)
- [lib/elliptic/curve/mont.js](lib/elliptic/curve/mont.js) - Montgomery curves
- [lib/elliptic/curves.js](lib/elliptic/curves.js) - Curve registry & presets

#### Protocol Implementations
- [lib/elliptic/ec/](lib/elliptic/ec/) - ECDSA implementation
  - [index.js](lib/elliptic/ec/index.js) - EC context
  - [key.js](lib/elliptic/ec/key.js) - Key pair management & signing/verification
  - [signature.js](lib/elliptic/ec/signature.js) - Signature encoding/decoding (DER, RAW)
- [lib/elliptic/eddsa/](lib/elliptic/eddsa/) - EdDSA implementation  
  - [index.js](lib/elliptic/eddsa/index.js) - EdDSA context
  - [key.js](lib/elliptic/eddsa/key.js) - EdDSA key operations
  - [signature.js](lib/elliptic/eddsa/signature.js) - EdDSA signature encoding

### Utilities
- [lib/elliptic/utils.js](lib/elliptic/utils.js) - Helper functions (encoding, assertion, random generation)

### Tests Organization
- [test/api-test.js](test/api-test.js) - Basic API instantiation
- [test/ecdsa-test.js](test/ecdsa-test.js) - ECDSA sign/verify tests
- [test/ed25519-test.js](test/ed25519-test.js) - EdDSA tests
- [test/ecdh-test.js](test/ecdh-test.js) - Key exchange tests
- [test/curve-test.js](test/curve-test.js) - Curve arithmetic tests

## Code Conventions & Patterns

### Module Pattern
All modules follow strict mode and CommonJS exports:
```javascript
'use strict';

var ModuleClass = function() { /* ... */ };
module.exports = ModuleClass;
```

### Dependencies
Key external dependencies:
- **bn.js** - BigNumber operations (all cryptographic arithmetic uses BN)
- **hash.js** - Hashing utilities
- **hmac-drbg** - Deterministic random generation for signatures
- **brorand** - Random bytes generation
- **minimalistic-assert** - Lightweight assertion

### Code Style (ESLint Rules)
- **Indentation**: 2 spaces
- **Quotes**: Single quotes (with `avoidEscape: true`)
- **Semicolons**: Always required
- **Spacing**: Spaces in parentheses, object literals; no whitespace before properties
- **Naming**: camelCase (enforced by ESLint)
- **Line endings**: Unix only (LF)
- **Array brackets**: Always use `[ ]` with spaces inside

See [.eslintrc.js](.eslintrc.js) for full rules.

### Common Patterns

#### BigNumber (BN) Usage
All curve operations use `bn.js` for arbitrary-precision arithmetic:
```javascript
var BN = require('bn.js');
var n = new BN('deadbeef', 16);
var result = n.mul(other).mod(prime);
```

#### Curve Types
Curves are dynamically instantiated as `short`, `edwards`, or `mont` types:
```javascript
var curve = new curve.short(options);  // for NIST, secp256k1
var curve = new curve.edwards(options); // for Ed25519
var curve = new curve.mont(options);   // for Curve25519
```

#### Encoding/Decoding
Points and signatures support multiple encoding formats:
- **Hex strings**: `'04...'` for points, `'3046...'` for signatures
- **Object notation**: `{ x: '...', y: '...' }` for points, `{ r: '...', s: '...' }` for ECDSA
- **Buffer format**: Node.js Buffer objects

#### Signature Formats
- **ECDSA**: DER encoding (standard) or RAW format `{ r, s }`
- **EdDSA**: 64-byte concatenation format

### Key Methods to Understand

#### EC (ECDSA)
- `ec.genKeyPair()` - Generate random key pair
- `ec.keyFromPrivate(priv, enc)` / `ec.keyFromPublic(pub, enc)` - Import keys
- `keyPair.sign(hash)` - Sign message hash
- `keyPair.verify(hash, signature)` - Verify signature
- `keyPair.getPublic(compact, enc)` - Export public key

#### EdDSA
- `eddsa.keyFromSecret(secret)` - Derive key from secret
- `eddsa.keyFromPublic(pub)` - Import public key
- Similar sign/verify interface to EC

### Important Notes & Pitfalls

1. **Hash Input Format**: `sign()` expects message hash as array or hex string, NOT the original message
   ```javascript
   var msgHash = [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ];
   var sig = key.sign(msgHash);  // ✓ Correct
   // NOT: key.sign('message')   // ✗ Wrong
   ```

2. **Curve Selection**: Always validate curve security at [SafeCurves](http://safecurves.cr.yp.to/) before use

3. **Deterministic Signing**: ECDSA uses RFC6979 (HMAC-DRBG) for deterministic random generation in signatures

4. **Infinity Point Checks**: Many validations check if a point equals the curve's point at infinity
   - Use `point.isInfinity()` to check
   - Operations like `G * n` (generator × order) should equal infinity

5. **Browser Bundling**: Gruntfile builds browser-compatible bundles via Browserify with `brfs` transform

6. **Performance**: 
   - Benchmarks available in [benchmarks/index.js](benchmarks/index.js)
   - Precomputed tables in [lib/elliptic/precomputed/](lib/elliptic/precomputed/) for speed optimization

## When Making Changes

- Always run `npm test` before committing (lints + tests)
- Add test cases in [test/](test/) for new features
- Follow ESLint rules; `npm run lint:fix` auto-corrects most issues
- BigNumber operations: use `bn.js` API, test with edge cases (0, 1, negative, large numbers)
- Document curve parameter selection logic and security assumptions
- Keep curve validation logic strict (verify G and n with `validate()` and point checks)

## File Mapping Quick Reference

| Purpose | File |
|---------|------|
| API entry | [lib/elliptic.js](lib/elliptic.js) |
| ECDSA signing | [lib/elliptic/ec/key.js](lib/elliptic/ec/key.js) |
| EdDSA signing | [lib/elliptic/eddsa/key.js](lib/elliptic/eddsa/key.js) |
| Signature encoding | [lib/elliptic/ec/signature.js](lib/elliptic/ec/signature.js) |
| Curve math | [lib/elliptic/curve/base.js](lib/elliptic/curve/base.js) |
| Test runner | [test/index.js](test/index.js) |
| ESLint config | [.eslintrc.js](.eslintrc.js) |
| Build script | [Gruntfile.js](Gruntfile.js) |
