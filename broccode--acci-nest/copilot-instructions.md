## acci-nest

> ALWAYS apply when defining entity relationships in MikroORM to ensure proper bidirectional relationships configuration and avoid duplicate field errors


# MikroORM Bidirectional Relationships

## Context

- Apply when defining entity relationships in MikroORM
- Use when setting up one-to-many, many-to-one, or many-to-many relationships
- Reference when debugging entity relationship errors like "Both entities are defined as owning sides"
- Follow when implementing multi-tenant entity relationships
- Check when getting duplicate field name errors

## Critical Rules

- In BIDIRECTIONAL relationships, ONE side must be the OWNING side, and the OTHER side must be the INVERSE side
- For ManyToOne/OneToMany relationships, the ManyToOne side is automatically the owning side
- For ManyToMany relationships, explicitly define the owning side with `{ owner: true }`
- The inverse side must use `{ mappedBy: 'propertyName' }` referencing the property on the other entity
- When using a relation (e.g. `@ManyToOne`) AND an explicit ID field, avoid duplicate field names with `{ fieldName: 'custom_field_name' }`
- For tenant relationships, use a custom `fieldName` to prevent conflicts with explicit `tenantId` properties
- Test entity definitions before implementing complex services to catch relationship errors early
- Never define both sides of a bidirectional relationship as owning sides

## Examples

<example>
// User entity (owning side of the relationship)
@Entity()
export class User extends BaseEntity {
  // ManyToOne side is automatically the owning side
  @ManyToOne(() => Tenant, { fieldName: 'tenant_reference_id' })
  tenant!: Tenant;

  @Property()
  tenantId!: string;

  // Explicitly mark as owning side in ManyToMany
  @ManyToMany(() => Role, role => role.users, { owner: true })
  roles = new Collection<Role>(this);
}

// Role entity (inverse side of the relationship)
@Entity()
export class Role extends BaseEntity {
  @ManyToOne(() => Tenant, { fieldName: 'tenant_reference_id' })
  tenant!: Tenant;

  @Property()
  tenantId!: string;

  // Mark as inverse side with mappedBy
  @ManyToMany(() => User, user => user.roles, { mappedBy: 'roles' })
  users = new Collection<User>(this);

  // This relationship has Role as the owning side
  @ManyToMany(() => Permission, permission => permission.roles, { owner: true })
  permissions = new Collection<Permission>(this);
}

// Permission entity (inverse side with Role)
@Entity()
export class Permission extends BaseEntity {
  // Mark as inverse side with mappedBy
  @ManyToMany(() => Role, role => role.permissions, { mappedBy: 'permissions' })
  roles = new Collection<Role>(this);
}
</example>

<example type="invalid">
// User entity - ERROR: Both sides are owning
@Entity()
export class User extends BaseEntity {
  // ERROR: Same column name conflict
  @ManyToOne(() => Tenant)
  tenant!: Tenant;

  @Property()
  tenantId!: string; // Conflicts with tenant relation

  // No owner specification
  @ManyToMany(() => Role, role => role.users)
  roles = new Collection<Role>(this);
}

// Role entity - ERROR: Both sides are owning
@Entity()
export class Role extends BaseEntity {
  // No owner specification or mappedBy
  @ManyToMany(() => User, user => user.roles)
  users = new Collection<User>(this);
  
  // Both sides defined as owning sides
  @ManyToMany(() => Permission, permission => permission.roles)
  permissions = new Collection<Permission>(this);
}

// Permission entity - ERROR: Both sides are owning
@Entity()
export class Permission extends BaseEntity {
  // No mappedBy option
  @ManyToMany(() => Role, role => role.permissions)
  roles = new Collection<Role>(this);
}</example> 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Broccode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
