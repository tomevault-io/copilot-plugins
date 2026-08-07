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

## Use `pytest` instead of `unittest`

In Sentry Python tests, prefer using `pytest` instead of `unittest`. This promotes consistency, reduces boilerplate, and leverages shared test setup logic defined in the factories.

For example, a diff that uses `pytest` instead of `unittest` would look like:

```diff
    -        self.assertRaises(ValueError, EffectiveGrantStatus.from_cache, None)
    +        with pytest.raises(ValueError):
    +            EffectiveGrantStatus.from_cache(None)
```

---
> Source: [sindhujaad/sentry__sentry-greptile__ace-code-review__PR3__20260725](https://github.com/sindhujaad/sentry__sentry-greptile__ace-code-review__PR3__20260725) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
