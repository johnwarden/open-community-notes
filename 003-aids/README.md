# Open Community Notes Anonymous ID (AID) Specification

This document describes the Anonymous ID (AID) system for the Community Notes service, including generation, validation, and security properties.

## Overview

Anonymous IDs (AIDs) are pseudonymous identifiers that allow users to create and rate Community Notes while maintaining privacy. AIDs are generated deterministically from user DIDs using the service's private key as a secret salt.

## AID Format

24-character base32-encoded strings (like PLC DIDs), representing the first 120 bits of a SHA256 hash.

## Security Properties

The AID generation provides the following security properties:

1. **Rainbow table resistant**: The service private key prevents pre-computation attacks, or from simply guessing who a note author or rater might be and simply hashing their DID to verify.
2. **Service-specific**: Different services with different keys produce different AIDs for the same user
3. **Deterministic**: Same inputs always produce the same AID
4. **Stable**: AIDs remain constant as long as the service private key is stable

## Service ID Space Separation

Since AIDs are generated per-service, the same user should have different AIDs on different Community Notes services.

However, there is no way that a third-party can validate generated AIDs (not knowing the secret salt of the service). A malicious service could intentionally use an AID corresponding to a user on a different service.

So it is essentially that when running the algorithm, AIDs are prefixed or namespaced (e.g. using the service DID).

## Private Key Management

**No Rotation**. The secret salt can't be rotated without breaking the mapping between DIDs and AIDs


