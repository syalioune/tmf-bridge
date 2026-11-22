# Draft User Journeys: TMF-Bridge

Marking: every detail not stated in the brief, addendum, original-brief extract or the user's facts is tagged [INFERRED]. Personas Claire and Marc exist in the source input; Inès and Tomás are invented [INFERRED].

## UJ-1: Inès adds a backend field to a TMF620 attribute and carries it to production

**Protagonist:** Inès, integration analyst at a mid-size operator [INFERRED: name and employer]. She understands the catalog data but is not a developer. Claire (Head of BSS Transformation) wants "just configuration".

**Steps**
1. Partners ask for a product family label on productOffering. Inès opens the minimal Mapping Studio against the mock catalog backend (backends are mocks in the first slice).
2. She maps the backend field `family_name` to a TMF620 attribute, with no code. [INFERRED: field and attribute names]
3. She runs a test call. The response shows the new value and its field lineage (backend source and transformation), and passes runtime OpenAPI validation.
4. She exports the mapping as structured text and pushes it to Git. [INFERRED: via a pull request reviewed by Marc, the solution architect]
5. After merge, the pipeline applies the configuration to production through the REST API, with no restart and no manual step. [INFERRED: pipeline triggered on merge]
6. Drift detection reports the running configuration matches Git.
7. The next TMF620 response in production carries the field.

**Where it lands:** a versioned, audited change delivered without developer help.

**Capabilities exercised:** minimal studio, mapping, field lineage, runtime OpenAPI validation, structured-text export, config as code via Git, REST API apply, no-restart configuration, drift detection.

## UJ-2: Tomás, a partner developer, consumes the exposed TMF620 API

**Protagonist:** Tomás, developer on a partner portal team [INFERRED: name and role]. He is a northbound API caller, not a user of the studio.

**Steps**
1. Tomás calls `GET /productOffering/{id}` using the official TMF620 definition [INFERRED: he codes against the published spec].
2. The response is schema-valid. It is validated at runtime against the official OpenAPI definition, and legacy internals do not leak.
3. It returns within the performance target. [INFERRED: target to be set in the PRD, per the brief]
4. Tomás tries a resource not yet configured, such as TMF622 productOrder. He gets `501` with a TMF Error, not a crash or an empty body.
5. Later the operator replaces the legacy catalog behind the bridge. [INFERRED: Inès re-maps the new backend through the UJ-1 flow]
6. Tomás's calls keep working. The contract is unchanged and he changes no code.

**Where it lands:** a trustworthy, spec-conformant API whose backend can change underneath it.

**Capabilities exercised:** TMF620 read path, runtime OpenAPI validation, performance metrics, 501 with TMF Error for unconfigured resources, mapping performed by the bridge, backend swap without contract change. This also supports the "built to leave" principle.
