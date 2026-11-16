# AWS IAM, Azure RBAC and Google Cloud IAM

A comparison brief, written from each vendor's documentation. Sources checked
2026-11-11 UTC.

Each cloud answers the same question: may this principal perform this action
on this resource? Authentication identifies the caller. Authorization decides
what the caller may do.

| Question | AWS IAM | Azure RBAC | Google Cloud IAM |
|---|---|---|---|
| What grants access? | Identity policies and resource policies, subject to applicable limits | A role assignment: principal + role definition + scope | An allow-policy binding: principal + role, with an optional condition |
| Who runs an application? | Usually an assumed IAM role with temporary credentials | Usually a managed identity or service principal | Usually a service account or federated workload identity |
| How is scope expressed? | Policy resources and conditions; accounts sit within an organization | Management group → subscription → resource group → resource | Organization → folder → project → resource |
| How does inheritance work? | Organization policies constrain member accounts; IAM policies name resources rather than inherit through an Azure-style resource tree | Role assignments at a parent scope apply below it | Parent allow policies add to grants on the resource |
| What restricts a grant? | Applicable explicit denies override allows; boundaries and organization policies limit rather than grant access | Applicable deny assignments can block granted actions; `NotActions` only subtracts from one role | Applicable deny policies override allows for supported permissions |

Sources: [AWS evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html),
[Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview),
[Google allow policies](https://docs.cloud.google.com/iam/docs/allow-policies),
[Google deny policies](https://docs.cloud.google.com/iam/docs/deny-overview).

## Three mistakes to avoid

**An AWS permissions boundary does not grant permission.** It caps what identity
policies can grant. Resource-policy evaluation has principal-specific exceptions;
check the actual request and principal instead of assuming every allow is intersected
with every boundary. [AWS evaluation details](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-denyallow.html)

**Azure resource management and data access are distinct.** Role definitions use
`Actions` for the control plane and `DataActions` for the data plane. Do not infer
permission to read stored data from the ability to manage its resource. `NotActions`
is not a deny: another role can grant the excluded action. Azure deny assignments
are a separate mechanism. [Role definitions](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-definitions),
[deny assignments](https://learn.microsoft.com/en-us/azure/role-based-access-control/deny-assignments)

**A Google child policy cannot erase a parent allow.** A narrower child allow
policy does not remove inherited grants. Remove or narrow the grant where it was
made, or use a supported deny policy. A service account is a principal; a role is
a collection of permissions. [Allow-policy inheritance](https://docs.cloud.google.com/iam/docs/allow-policies)

## One interview example

An application needs to read one storage container. In AWS, grant its role the
needed object-read action for the intended object resources. In Azure, assign a
data-reading role to its managed identity at the intended container scope. In
Google Cloud, bind an object-reading role to its service account on the intended
bucket. In all three, check inherited grants and applicable restrictions, then
verify a permitted read and a denied read outside that scope.

Explain each model without notes, then answer: why can a narrow child assignment
leave a broader inherited grant intact? Why can managing storage differ from
reading its contents? These questions test the mechanism, not product vocabulary.
