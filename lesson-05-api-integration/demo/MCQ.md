## Question 1

**Scenario:**  
You have successfully tested your agent locally, where you used
`gcloud auth application-default login` to access the Secret Manager. You are
now deploying the agent to a Google Cloud Run service. You want to ensure the
deployed agent can still retrieve the API key without modifying the code or
embedding credentials.

**Question:**  
What configuration is required in the Cloud Run environment to allow
`secretmanager.SecretManagerServiceClient()` to function correctly?

**Options:**

- **A)** Copy your local `application_default_credentials.json` file into the
  container image.
- **B)** Ensure the Service Account attached to the Cloud Run service has the
  `Secret Manager Secret Accessor` IAM role.
- **C)** Hardcode the API key into the `tools.py` file before deployment.
- **D)** Run `gcloud auth application-default login` as a startup command inside
  the container.

**Rationale:**

- **A)** Wrong — Copying user credentials to a production environment is a
  security risk and bad practice.
- **B)** Correct — In Google Cloud, ADC automatically uses the attached Service
  Account. This account must have the specific IAM role to access the secret.
- **C)** Wrong — Hardcoding keys defeats the purpose of using Secret Manager.
- **D)** Wrong — Interactive login flows are not suitable for headless
  production environments.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Securely retrieve API keys from Google Cloud Secret
Manager using Application Default Credentials (ADC).

## Question 2

**Scenario:**  
A developer accidentally pushes the `tools.py` file to a public GitHub
repository. The file contains the line:  
`SECRET_ID = "places-api-key"`  
but does not contain the actual API key string.

**Question:**  
From a security perspective, how would you evaluate the risk of this specific
exposure?

**Options:**

- **A)** Critical Risk — The `SECRET_ID` is sufficient for anyone to retrieve
  the key from Secret Manager.
- **B)** Low Risk — The `SECRET_ID` is just a reference name; an attacker would
  still need an authenticated Google Cloud account with specific IAM permissions
  to access the actual secret value.
- **C)** No Risk — The `SECRET_ID` is encrypted by default.
- **D)** High Risk — This ID allows bypassing the Application Default
  Credentials check.

**Rationale:**

- **A)** Wrong — The ID alone does not grant access; authentication and
  authorization are required.
- **B)** Correct — The security model relies on IAM. Only identities (
  Users/Service Accounts) with explicit permission on that project and secret
  can access it. Knowing the name isn't enough.
- **C)** Wrong — The ID itself isn't encrypted (it's a string in code), but it's
  not the secret data.
- **D)** Wrong — ADC cannot be bypassed just by knowing a resource name.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Securely retrieve API keys from Google Cloud Secret
Manager using Application Default Credentials (ADC).

## Question 3

**Scenario:**  
Your agent is handling a high volume of requests. You notice that the costs for
the Google Routes API are higher than expected, and the latency is high. You
review the `get_route_between_places` function in `tools.py`.

**Question:**  
Which element of the API request configuration is most critical for controlling
the response size and associated costs/latency?

**Options:**

- **A)** The `X-Goog-Api-Key` header.
- **B)** The `travelMode` parameter in the JSON body.
- **C)** The `X-Goog-FieldMask` header.
- **D)** The `Content-Type` header.

**Rationale:**

- **A)** Wrong — This is for authentication, not data shaping.
- **B)** Wrong — This changes the calculation logic, not necessarily the amount
  of data returned (though different modes might have different data).
- **C)** Correct — The Field Mask explicitly tells the API to return *only* the
  requested fields (e.g., `routes.distanceMeters`). Without it, the API might
  return the full route geometry and steps, significantly increasing payload
  size and cost.
- **D)** Wrong — This specifies the format of the request body.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Perform HTTP requests (POST) to interact with RESTful
APIs.

## Question 4

**Scenario:**  
The agent calls the `get_place_details` tool with the argument
`place1="Atlantis"`. The Google Places API returns a successful response object
but finds no matching places (`response.places` is empty).

**Question:**  
Based on the logic in `tools.py`, what will the `get_details` inner function
return?

**Options:**

- **A)** It raises a `ValueError`.
- **B)** It returns an empty dictionary `{}`.
- **C)** It returns `None`.
- **D)** It retries the request automatically.

**Rationale:**

- **A)** Wrong — The code checks `if response and response.places`.
- **B)** Wrong — The fallback return statement is `return None`.
- **C)** Correct — If the `if` condition fails (no places found), the function
  proceeds to the final line: `return None`.
- **D)** Wrong — There is no retry logic implemented in the provided snippet.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Construct complex tools that combine data from multiple
API endpoints.

## Question 5

**Scenario:**  
You are setting up the project on a new development machine. You have installed
the requirements and set the `GOOGLE_CLOUD_PROJECT` environment variable.
However, when you run the agent, it crashes with
`google.auth.exceptions.DefaultCredentialsError`.

**Question:**  
What is the most likely cause of this error?

**Options:**

- **A)** The Google Maps Places API is not enabled in the Cloud Console.
- **B)** You have not run `gcloud auth application-default login` to generate
  the local credentials file.
- **C)** The `places-api-key` secret does not exist in Secret Manager.
- **D)** The `requests` library is outdated.

**Rationale:**

- **A)** Wrong — API enablement issues usually cause 403 or 400 errors from the
  API, not a local credential error.
- **B)** Correct — `DefaultCredentialsError` indicates that the Google Cloud
  client library cannot find any valid credentials in the standard locations (
  like the file created by `gcloud auth application-default login`).
- **C)** Wrong — If the secret didn't exist, you would get a `NotFound` or
  `PermissionDenied` error *after* authentication succeeded.
- **D)** Wrong — Unrelated to Google Auth.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Securely retrieve API keys from Google Cloud Secret
Manager using Application Default Credentials (ADC).

## Question 6

**Scenario:**  
You want to extend the `get_place_details` function to also return the
international phone number for a location.

**Question:**  
In addition to extracting the field from the response object, what specific
modification must you make to the API request setup in the code?

**Options:**

- **A)** Update the `fields` list to include `places.internationalPhoneNumber`
  so it is added to the field mask.
- **B)** Switch from using `PlacesClient` to `requests.post` to allow extra
  fields.
- **C)** Add a new header `X-Goog-Phone-Number: true`.
- **D)** Grant the "Phone Number Viewer" IAM role to the service account.

**Rationale:**

- **A)** Correct — The Places API (v1) requires you to explicitly request fields
  via the field mask. If you don't ask for it in the mask, the API won't return
  it, even if it exists.
- **B)** Wrong — The client library supports field masks; switching libraries
  isn't necessary.
- **C)** Wrong — This is not a valid API header.
- **D)** Wrong — Data access is controlled by the API Key and Field Mask, not
  granular IAM roles for specific fields.

**Difficulty:** Advanced  
**Cognitive Level:** Application  
**Learning Objective:** Integrate external APIs (Google Places and Routes) into
ADK tools.

## Question 7

**Scenario:**  
Your code successfully retrieves an API Key from Secret Manager. However, when
`get_route_between_places` executes, the `requests.post` call returns a
`403 Forbidden` error. You confirm that the Service Account has full Secret
Manager permissions.

**Question:**  
What is the most probable configuration issue?

**Options:**

- **A)** The Service Account is missing the `Compute Engine Viewer` role.
- **B)** The API Key stored in Secret Manager has API restrictions that do not
  include the "Google Routes API".
- **C)** The `X-Goog-FieldMask` header is missing, causing a permission denial.
- **D)** ADC has failed to refresh the token.

**Rationale:**

- **A)** Wrong — Compute Engine roles are irrelevant to Maps APIs.
- **B)** Correct — Once the code has the key, it acts as that key. If the key
  itself is restricted (a best practice) but is missing the Routes API in its
  allowed list, the Routes API will reject it with a 403.
- **C)** Wrong — Missing field masks usually result in default behavior (
  returning all fields) or a 400 Bad Request, not a 403.
- **D)** Wrong — The API Key is a static string; it doesn't "refresh" like an
  OAuth token (though it can be rotated, the error implies the current one is
  invalid for this purpose).

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Integrate external APIs (Google Places and Routes) into
ADK tools.

## Question 8

**Scenario:**  
In `tools.py`, the `get_route_between_places` function uses the `requests`
library to manually construct an HTTP POST request, whereas `get_place_details`
uses the `google.maps.places_v1` client library.

**Question:**  
What is a direct technical consequence of using `requests` for the Routes API
integration as shown in this code?

**Options:**

- **A)** It is impossible to use Application Default Credentials (ADC) with
  `requests`.
- **B)** You must manually handle the construction of headers (like
  `X-Goog-Api-Key`) and the serialization of the JSON payload.
- **C)** The `requests` library provides built-in retries for 5xx errors, which
  the client library lacks.
- **D)** You cannot use Field Masks with raw HTTP requests.

**Rationale:**

- **A)** Wrong — You *can* use ADC with requests (by getting a token), but here
  we are using an API Key, so ADC is only used to get the secret.
- **B)** Correct — Using a client library abstracts away the HTTP details. Using
  `requests` requires the developer to explicitly set headers, content types,
  and structure the JSON body according to the API spec.
- **C)** Wrong — `requests` does *not* retry by default; you have to implement
  that (e.g., using `HTTPAdapter`). Client libraries often have built-in retry
  logic.
- **D)** Wrong — Field masks are just headers or parameters; they work fine with
  raw HTTP.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Perform HTTP requests (POST) to interact with RESTful
APIs.
