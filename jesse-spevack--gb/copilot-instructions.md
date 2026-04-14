## gb

> Description of the proper enum syntax

- **Standard Syntax:**
  - Use the `enum :attribute_name, { key: value, ... }` syntax for defining enums.
  - This maps string/symbol keys to underlying database values (often integers).

  ```ruby
  # ✅ In app/models/student_work.rb
  class StudentWork < ApplicationRecord
    # ...
    enum :status, { pending: 0, processing: 1, completed: 2, failed: 3 }
    # ...
  end
  ```

- **Database Column:**
  - The corresponding database column should typically be an integer.
  - Define a default value in the migration using the integer mapping.
  - Add a database index for performance if the status is frequently queried.

  ```ruby
  # ✅ In db/migrate/..._create_student_works.rb
  class CreateStudentWorks < ActiveRecord::Migration[8.0]
    def change
      create_table :student_works do |t|
        # ...
        t.integer :status, null: false, default: 0 # Default maps to :pending
        # ...
        t.timestamps
      end
      add_index :student_works, :status
    end
  end
  ```

- **Default Value:**
  - It is sufficient and clearer to rely on the database default set in the migration, especially for new records.

- **Helper Methods:**
  - Defining an enum automatically creates helpful query scopes (e.g., `StudentWork.pending`) and instance methods (e.g., `student_work.pending?`, `student_work.processing!`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jesse-spevack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
