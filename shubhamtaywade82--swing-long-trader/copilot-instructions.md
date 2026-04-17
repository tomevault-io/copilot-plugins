## swing-long-trader

> - ALWAYS use the Ruby documentation convention of `.` (or `::`) when referring to a class method's name

# Better Specs - RSpec Testing Guidelines

## Describe Your Methods
- ALWAYS use the Ruby documentation convention of `.` (or `::`) when referring to a class method's name
- ALWAYS use `#` when referring to an instance method's name
- Example: `describe '.authenticate'` for class methods, `describe '#admin?'` for instance methods
- NEVER use vague descriptions like 'the authenticate method for User' or 'if the user is an admin'

## Use Contexts
- ALWAYS use contexts to make tests clear and well organized
- ALWAYS start context descriptions with 'when', 'with', or 'without'
- Example: `context 'when logged in'`, `context 'with valid params'`, `context 'without authentication'`
- NEVER write conditions directly in test descriptions like 'has 200 status code if logged in'

## Keep Descriptions Short
- ALWAYS keep spec descriptions under 40 characters
- ALWAYS split longer descriptions using contexts
- ALWAYS use `is_expected.to` for one-liner expectations to keep descriptions concise
- Example: Instead of 'has 422 status code if an unexpected params will be added', use:
  ```ruby
  context 'when not valid' do
    it { is_expected.to respond_with 422 }
  end
  ```

## Single Expectation Test
- ALWAYS use one expectation per test in isolated unit specs
- ALLOW multiple expectations in non-isolated tests (DB, external services, end-to-end tests) for performance
- ALWAYS split tests with multiple expectations into separate tests when possible
- Example (isolated):
  ```ruby
  it { is_expected.to respond_with_content_type(:json) }
  it { is_expected.to assign_to(:resource) }
  ```
- Example (not isolated):
  ```ruby
  it 'creates a resource' do
    expect(response).to respond_with_content_type(:json)
    expect(response).to assign_to(:resource)
  end
  ```

## Test All Possible Cases
- ALWAYS test valid, edge, and invalid cases
- ALWAYS think of all possible inputs and test them
- ALWAYS test edge cases like: resource not found, resource not owned, missing required fields, etc.
- Example:
  ```ruby
  describe '#destroy' do
    context 'when resource is found' do
      it 'responds with 200'
      it 'shows the resource'
    end
    context 'when resource is not found' do
      it 'responds with 404'
    end
    context 'when resource is not owned' do
      it 'responds with 404'
    end
  end
  ```

## Expect vs Should Syntax
- ALWAYS use the `expect` syntax, NEVER use `should`
- ALWAYS use `is_expected.to` for one-liner expectations with implicit subject
- NEVER use `should` or `should_not` in expectations
- Example:
  ```ruby
  # Good
  expect(response).to respond_with_content_type(:json)
  context 'when not valid' do
    it { is_expected.to respond_with 422 }
  end

  # Bad
  response.should respond_with_content_type(:json)
  context 'when not valid' do
    it { should respond_with 422 }
  end
  ```

## Use Subject
- ALWAYS use `subject{}` to DRY up tests related to the same subject
- ALWAYS use named subjects when helpful: `subject(:hero) { Hero.first }`
- Example:
  ```ruby
  subject { assigns('message') }
  it { is_expected.to match /it was born in Billville/ }
  ```

## Use let and let!
- ALWAYS use `let` instead of `before` blocks to create instance variables
- ALWAYS use `let` for lazy-loaded variables (loaded only when first used)
- ALWAYS use `let!` when you need the variable defined immediately (e.g., for database queries or scopes)
- NEVER use `@instance_variable` in tests - use `let` instead
- Example:
  ```ruby
  # Good
  let(:resource) { FactoryBot.create :device }
  let(:type) { Type.find resource.type_id }

  # Bad
  before { @resource = FactoryBot.create :device }
  before { @type = Type.find @resource.type_id }
  ```

## Mock or Not to Mock
- ALWAYS prefer testing real behavior when possible
- NEVER overuse mocks - they make specs faster but are difficult to use correctly
- ALWAYS use mocks sparingly and only when necessary (e.g., external services, expensive operations)
- ALWAYS understand mocks well before using them
- Example:
  ```ruby
  context "when not found" do
    before do
      allow(Resource).to receive(:where).with(created_from: params[:id])
        .and_return(false)
    end
    it { is_expected.to respond_with 404 }
  end
  ```

## Create Only the Data You Need
- ALWAYS create only the minimum data needed for each test
- NEVER create dozens of records unless absolutely necessary
- ALWAYS consider test suite performance when creating test data
- Example:
  ```ruby
  describe "User" do
    describe ".top" do
      before { FactoryBot.create_list(:user, 3) }
      it { expect(User.top(2)).to have(2).item }
    end
  end
  ```

## Use Factories and Not Fixtures
- ALWAYS use FactoryBot (or similar) instead of fixtures
- ALWAYS use factories to reduce verbosity when creating test data
- NEVER use fixtures - they are difficult to control
- Example:
  ```ruby
  # Good
  user = FactoryBot.create :user

  # Bad
  user = User.create(
    name: 'Genoveffa',
    surname: 'Piccolina',
    city: 'Billyville',
    birth: '17 Agoust 1982',
    active: true
  )
  ```

## Easy to Read Matchers
- ALWAYS use readable matchers
- ALWAYS check available RSpec matchers before writing custom expectations
- ALWAYS use `expect { }.to raise_error` instead of `lambda { }.to raise_error`
- Example:
  ```ruby
  # Good
  expect { model.save! }.to raise_error Mongoid::Errors::DocumentNotFound

  # Bad
  lambda { model.save! }.to raise_error Mongoid::Errors::DocumentNotFound
  ```

## Shared Examples
- ALWAYS use shared examples to DRY up test suites when you see code duplication
- ALWAYS use `it_behaves_like` for shared behavior
- ALWAYS consider shared examples for controllers and similar patterns
- Example:
  ```ruby
  describe 'GET /devices' do
    let!(:resource) { FactoryBot.create :device, created_from: user.id }
    let!(:uri) { '/devices' }

    it_behaves_like 'a listable resource'
    it_behaves_like 'a paginable resource'
    it_behaves_like 'a searchable resource'
    it_behaves_like 'a filterable list'
  end
  ```

## Test What You See
- ALWAYS focus on testing models and application behavior (integration tests)
- ALWAYS prefer integration tests over controller tests when possible
- ALWAYS test the actual behavior users will see
- NEVER add unnecessary complexity testing controllers if integration tests cover the behavior

## Don't Use Should
- ALWAYS use third person present tense in test descriptions
- NEVER use 'should' in test descriptions
- ALWAYS use the new expectation syntax
- Example:
  ```ruby
  # Good
  it 'does not change timings' do
    expect(consumption.occur_at).to eq(valid.occur_at)
  end

  # Bad
  it 'should not change timings' do
    consumption.occur_at.should == valid.occur_at
  end
  ```

## Stubbing HTTP Requests
- ALWAYS stub external HTTP requests using WebMock or VCR
- ALWAYS stub requests in `before` blocks within appropriate contexts
- NEVER rely on real external services in tests
- Example:
  ```ruby
  context "with unauthorized access" do
    let(:uri) { 'http://api.example.com/types' }
    before { stub_request(:get, uri).to_return(status: 401, body: fixture('401.json')) }

    it "gets a not authorized notification" do
      page.driver.get uri
      expect(page).to have_content 'Access denied'
    end
  end
  ```

## Test Organization
- ALWAYS organize tests logically with clear contexts
- ALWAYS group related tests using nested `describe` blocks
- ALWAYS keep test structure flat and readable
- ALWAYS separate setup (mocks, data creation) from assertions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shubhamtaywade82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
