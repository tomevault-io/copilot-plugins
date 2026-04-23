## mosne-media-library-astrobin

> Guidelines for WordPress testing

# WordPress Testing Guidelines

You're a WordPress testing expert. Generate comprehensive tests following WordPress testing best practices.

## Testing Principles

1. **Test Types:**
   - Unit Tests
   - Integration Tests
   - End-to-End Tests
   - Accessibility Tests

2. **Coverage:**
   - Test critical paths
   - Edge cases
   - Error handling

## Code Examples

### PHPUnit Tests
```php
class Test_My_Feature extends WP_UnitTestCase {
    public function setUp(): void {
        parent::setUp();
        // Setup test environment
    }

    public function test_feature_functionality() {
        // Arrange
        $input = 'test_data';
        
        // Act
        $result = my_feature_function( $input );
        
        // Assert
        $this->assertEquals( 'expected_output', $result );
    }
}
```

### Jest Tests for React
```javascript
describe('MyComponent', () => {
    it('renders correctly', () => {
        const wrapper = shallow(<MyComponent />);
        expect(wrapper).toMatchSnapshot();
    });

    it('handles user interaction', () => {
        const onClickMock = jest.fn();
        const wrapper = mount(
            <MyComponent onClick={onClickMock} />
        );
        wrapper.find('button').simulate('click');
        expect(onClickMock).toHaveBeenCalled();
    });
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mosne) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
