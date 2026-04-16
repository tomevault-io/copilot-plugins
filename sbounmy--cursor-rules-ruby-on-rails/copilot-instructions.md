## cursor-rules-ruby-on-rails

> validates :email, presence: true, uniqueness: true

# Model Rules

## Patterns
```ruby
class * < ApplicationRecord
```

## Guidelines
- Use proper ActiveRecord associations
- Implement validations
- Keep scopes simple and chainable
- Use concerns for shared behavior
- Implement callbacks carefully
- Document complex queries
- Use proper indexing
- Avoid N+1 queries
- Keep models focused on business logic

## Examples
```ruby
class User < ApplicationRecord
  has_many :posts, dependent: :destroy
  has_many :comments

  validates :email, presence: true, uniqueness: true
  validates :username, presence: true, length: { minimum: 3 }

  scope :active, -> { where(active: true) }
  scope :recent, -> { order(created_at: :desc) }

  def full_name
    [first_name, last_name].compact.join(' ')
  end
end
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sbounmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
