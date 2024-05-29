## Installation/Verification:
- [Install Vault](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install#install-vault)
- [Verifying the Installation](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install#verifying-the-installation)

## Fork the amplify repo https://github.com/OpenSourceFellows/amplify
- [Fork a repository](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)

## Log into Cluster UI and obtain token:

1) Log into the [Cluster](https://vault-public-vault-22deb760.8ee49bbe.z1.hashicorp.cloud:8200) using the username and password provided. The namespace you will use will be your username: 
<img width="598" alt="Screenshot 2024-05-29 at 2 46 57 PM" src="https://github.com/eprice555/amplify_cicd_test/assets/99197915/9ea6bd4f-2c12-4d8d-a0ca-90d7dc782ee0">

2) Next select the user icon at the top left and click "Copy token". Save this output as we will need it later in the exercise:
<img width="1490" alt="Screenshot 2024-05-29 at 2 45 08 PM" src="https://github.com/eprice555/amplify_cicd_test/assets/99197915/30ed6596-c3cb-4a8e-91e9-26e5a6a6ee76">


## Configure Access

1) Create three environment variables: $VAULT_ADDR, $VAULT_TOKEN, and $VAULT_NAMESPACE on your local machine (examples below):
   - export VAULT_ADDR=https://vault-public-vault-22deb760.8ee49bbe.z1.hashicorp.cloud:8200
   - export VAULT_TOKEN=(paste the token value you copied previously)
   - export VAULT_NAMESPACE=admin/(your username)
  
2) Verify the server is running via the `vault status` command:
   - vault status
  
3) Login to the vault:
   - vault login

## Configure JWT Auth

#### *Enable JWT Auth*
   * Enable the JWT auth method, and use write to apply the configuration to your Vault. For oidc_discovery_url and bound_issuer parameters, use https://token.actions.githubusercontent.com. These parameters allow the Vault server to verify  the received JSON Web Tokens (JWT) during the authentication process.

    vault auth enable jwt

#### *Configure Auth method*

    vault write auth/jwt/config \
    bound_issuer="https://token.actions.githubusercontent.com" \
    oidc_discovery_url="https://token.actions.githubusercontent.com"

#### *Configure roles to group different policies together. If the authentication is successful, these policies are attached to the resulting Vault access token*

    vault write auth/jwt/role/demo -<<EOF
    {
     "role_type": "jwt",
     "user_claim": "workflow",
     "bound_claims": {
     "repository": "enter the forked username/repository here"
    },
    "policies": ["app-policy"],
    "ttl": "10m"
    }
    EOF


## Secret Engines
* Secrets engines are Vault components which store, generate or encrypt secrets
* Types of Engines - KV store, dynamic creds, Encryption as service
* Secret engines are plugins that need to be enabled, Community, Custom etc
* Types of secrets engines
    1. Ldap
    2. Databases
    3. KV engine

## Demo for `vault secrets engine - KV`  
#### *Enable engine*

    vault secrets enable -path=secrets/hashi-corp-hackpod kv-v2

#### *Add Static secrets*

     vault kv put -mount=secrets/hashi-corp-hackpod/SECRETE_VARIABLE api-key=supersecret
     vault kv put -mount=secrets/hashi-corp-hackpod/SECRETE_VARIABLE api-key=supersecret


We will need to create the following secrets, modify the example above to fit that:

TEST_AUTH0_DOMAIN = "https://amplify-app-production.herokuapp.com/"
TEST_AUTH0_AUDIENCE = "dev-3huf08ov.us.auth0.com"
TEST_STRIPE_SECRET_KEY = "sk_test_51IravfFqipIA40A3FOW6EzlXlJiXjL9V0FXKfb9n7cxh25Ww9QMA9aWwCzTSQscBOQFcB7s1TI6UCtW1JG83Hz1z000Sg2vSIr"
TEST_CICERO_API_KEY = "4f762d4fd380bd38746254e501ef9ceb0153576f"
TEST_LOB_API_KEY = test_eee78731c29fe718a5322e0cd51e9a02f0e"

#### Read Static secrets

    vault kv get -mount=secrets/hashi-corp-hackpod/ ait-12345/db
    vault kv get -mount=secrets/hashi-corp-hackpod/ ait-56789/db


## Vault Policies
* Policies provide a declarative way to grant or forbid access to certain paths and operations in Vault
* Policies are deny by default, so an empty policy grants no permission in the system

![Pollicyworkflow](https://developer.hashicorp.com/_next/image?url=https%3A%2F%2Fcontent.hashicorp.com%2Fapi%2Fassets%3Fproduct%3Dvault%26version%3Drefs%252Fheads%252Frelease%252F1.15.x%26asset%3Dwebsite%252Fpublic%252Fimg%252Fvault-policy-workflow.svg%26width%3D669%26height%3D497&w=1920&q=75)

## Demo for `vault policy`  
#### Create Policy

    #### *Configure a policy that only grants access to the specific paths your workflows will use to retrieve secrets*

    tee app-policy.hcl <<EOF
    path "secrets/hashi-corp-hackpod/*"
    {  
    capabilities = ["read"]
    }
    EOF



## Modify the following actions from your forked resporitory to retrieve the secrets we created and use them:

- .github/workflows/unit-tests.yml
- .github/workflows/integration-tests.yaml

## Sample GitHub Pipeline

        uses: hashicorp/vault-action@v2.4.0
        with:
          url: https://vault-public-vault-22deb760.8ee49bbe.z1.hashicorp.cloud:8200
          role: demo
          method: jwt
          namespace: "yournamespace"


## Test the updated workflows to confirm they are retrieving secrets from the vault


If you run into any issues please let us know!
