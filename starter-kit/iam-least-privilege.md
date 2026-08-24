# IAM Least Privilege Design — KijaniKiosk

## 1. Application Task

**Task:** The KijaniKiosk backend service needs to **read and write product images** to a specific cloud storage bucket (`kijanikiosk-product-images`).

Rather than creating a broad administrator role for the backend service, we define exactly what it needs to do its job and nothing more. This is the starting point of least privilege design — you cannot scope permissions correctly until the task itself is specific.

| Attribute | Detail |
| --- | --- |
| **Component** | Backend application service (e.g. product listing API) |
| **Resource** | `kijanikiosk-product-images` storage bucket |
| **Action needed** | Read and write (upload/retrieve) objects |
| **Action NOT needed** | Delete objects, manage users, modify IAM, access other buckets |

---

## 2. Required Permissions

The permissions below map directly to what the task in Section 1 requires — no more, no less.

**Allowed:**

- Read objects from `kijanikiosk-product-images`
- Write (upload) objects to `kijanikiosk-product-images`

**Explicitly NOT allowed:**

- Delete objects
- Create or manage IAM users/roles
- Modify bucket policies or permissions
- Access any other bucket or resource in the account
- List or manage billing, compute, or networking resources

---

## 3. IAM Policy (JSON)

This policy is written in AWS IAM JSON syntax as a reference example. It is not deployed — it exists here to demonstrate the reasoning in a concrete, reviewable format.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadWriteProductImagesOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::kijanikiosk-product-images/*"
    }
  ]
}
```

**Why the policy is written this way:**

- `Action` is limited to `GetObject` and `PutObject` only — there is no `DeleteObject`, no `PutBucketPolicy`, no `iam:*` action of any kind.
- `Resource` is scoped to a single named bucket (`kijanikiosk-product-images`) and not `"*"` (which would mean "every bucket in the account").
- There is only **one** `Sid` (statement) because there is only one task. Adding more permissions "just in case" would break the least privilege principle even if those permissions were never misused.

---

## 4. Documentation

### Who / what receives the role

The role is attached to the **backend application service** (e.g. the product listing microservice), not to a human user. This is a service-to-resource permission, not a person-to-resource permission — a distinction worth stating explicitly, since IAM design differs slightly depending on whether the identity is a human or a machine.

### What task it performs

The service uploads product images when a vendor adds a new product, and retrieves those images when a customer views the product catalog. That is the entire scope of its interaction with cloud storage.

### Which permissions it needs

Only `s3:GetObject` and `s3:PutObject`, scoped to the `kijanikiosk-product-images` bucket. These two actions are sufficient to cover both directions of the task (write on upload, read on display).

### Why unnecessary permissions are excluded

- **No delete access:** the backend has no legitimate reason to remove product images; deletion (if ever needed) should go through a separate, more tightly audited process — not the everyday application role.
- **No IAM management access:** if this service were ever compromised (e.g. through a vulnerability in the application code), an attacker with IAM-modifying permissions could create new admin users or escalate their own access. Removing that permission entirely closes off that entire attack path.
- **No access to other buckets:** the backend has nothing to do with, for example, billing data or internal logs stored elsewhere. Scoping to a single bucket means a compromised backend can only affect product images — the "blast radius" of any breach is contained.

### How the design follows least privilege

Least privilege means granting the **minimum access required to perform a specific task, and nothing more** — access is earned by need, not convenience. This design follows that principle in three concrete ways:

1. **Task-first design** — permissions were derived from a specific, named task (read/write product images), not copied from a general-purpose template.
2. **Resource scoping** — the policy targets one named bucket via its ARN, not a wildcard `*` that would apply account-wide.
3. **Action scoping** — only the two actions actually used by the application (`GetObject`, `PutObject`) are allowed; every other action, including seemingly "harmless" ones like listing buckets, is left out by default.
If KijaniKiosk grows and the backend needs new capabilities (e.g. generating thumbnails, which might need `s3:DeleteObject` for cleanup), that permission should be added deliberately and reviewed at that time — not granted upfront "in case it's needed later." 