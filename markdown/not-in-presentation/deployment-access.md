# Section 6: Getting Access to Your Deployment

## Slide

---

### Getting Access to Your Deployment

**Dev deployment** = GCP project (e.g. `spark-dev-xyz`)

How to get access:

- **Shared team project** — ask your team/TL which GCP projects your team owns
- **Specific project** — find the owners in the GCP IAM console and request access

**Checking if you have access:**

- GCP IAM page: `console.cloud.google.com/iam-admin/iam?project=spark-dev-xyz`
- You might already have access through a Google group — check at `groups.google.com/my-groups`

**Prod?** Different process entirely — that goes through Bastion.

---

## Speech

So before you can actually work on a deployment, you need access to the GCP project behind it. The naming is usually one-to-one — `dev-xyz` deployment lives in the `spark-dev-xyz` project.

Most of the time, your team already has shared projects. Ask your TL or look around — there's a good chance you're already on a Google group that has access and you just don't know it. You can check your groups at `groups.google.com/my-groups`. If you need a specific project that your team doesn't own, go to the IAM console for that project, find whoever's listed as owner, and ask them to add you.

To verify whether you have access, go to the IAM page for the project — `console.cloud.google.com/iam-admin/iam?project=` and then the project name. If you can see the page, you have access. If not, you don't.

For prod deployments, none of this applies — that's a separate process through Bastion.
