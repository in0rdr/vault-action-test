# Vault Action Test

This repo contains a test for the [vault-action](https://github.com/hashicorp/vault-action) GitHub action.

## Vault Server Configuration

````bash
vault policy write test-role - <<EOF
# Allow creating limited child token
# This is required when using JWT auth inside workflow
path "auth/token/create" {
    capabilities = ["create", "update"]
}
EOF

vault auth enable -path=jwt-test jwt

vault write auth/jwt-test/config \
        oidc_discovery_url="https://token.actions.githubusercontent.com" \
        bound_issuer="https://token.actions.githubusercontent.com" \
        default_role="test-role"

vault write auth/jwt-test/role/test-role -<<EOF
{
  "user_claim": "actor",
  "groups_claim": "repository_owner",
  "role_type": "jwt",
  "policies": "test-role",
  "ttl": "1h",
  "bound_claims": {
    "repository": "in0rdr/vault-action-test",
    "job_workflow_ref": "in0rdr/vault-action-test/.github/workflows/vault.yml@refs/heads/main"
  },
  "bound_audiences": "https://github.com/in0rdr"
}
EOF
````
