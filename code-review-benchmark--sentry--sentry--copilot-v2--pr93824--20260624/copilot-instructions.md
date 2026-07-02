## python-tests

> Guidelines for writing python tests


# Python tests


## Use factories instead of directly calling `Model.objects.create`

In Sentry Python tests, prefer using factory methods from `sentry.testutils.factories.Factories` @factories.py or fixture methods (e.g., `self.create_model`) provided by base classes like `sentry.testutils.fixtures.Fixtures` @fixtures.py  instead of directly calling `Model.objects.create`. This promotes consistency, reduces boilerplate, and leverages shared test setup logic defined in the factories.

For example, a diff that uses a fixture instead of the directly calling `Model.objects.create`  would look like:

```diff
    -        direct_project = Project.objects.create(
    -            organization=self.organization,
    -            name="Directly Created",
    -            slug="directly-created"
    -        )
    +        direct_project = self.create_project(
    +            organization=self.organization,
    +            name="Directly Created",
    +            slug="directly-created" # Note: Ensure factory args match
    +        )
```

---
> Source: [code-review-benchmark/sentry__sentry__copilot-v2__PR93824__20260624](https://github.com/code-review-benchmark/sentry__sentry__copilot-v2__PR93824__20260624) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
