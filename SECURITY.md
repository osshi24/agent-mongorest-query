# 🔒 Security Best Practices

## Environment Variables

This project uses environment variables to store sensitive information. **NEVER hardcode secrets in source code.**

### Setup

1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```

2. Generate a strong SECRET_KEY:
   ```bash
   # Using OpenSSL
   openssl rand -hex 32
   
   # Using Node.js
   node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
   ```

3. Add the generated key to your `.env` file:
   ```
   SECRET_KEY=your-generated-key-here
   ```

4. **IMPORTANT**: Never commit `.env` file to Git (already in `.gitignore`)

### Why This Matters

- ❌ Hardcoded secrets can be exposed in version control history
- ❌ Public repositories expose secrets to everyone
- ❌ Weak secrets (like '1234567890') are easily guessed
- ✅ Environment variables keep secrets separate from code
- ✅ Each environment (dev/staging/prod) can use different secrets
- ✅ Secrets can be rotated without code changes

## Reporting Security Issues

If you discover a security vulnerability, please email: [your-security-email]

Do NOT create public issues for security vulnerabilities.
