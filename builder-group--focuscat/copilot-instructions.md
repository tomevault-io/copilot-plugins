## react

> React component patterns and conventions


# React Style Guide

React component patterns and conventions for our codebase.

## Component Definition

- Always use `React.FC<TProps>` pattern for components
- Always destructure props at the top with `const { ... } = props`
- Always define interface at the bottom of the file with `T` prefix
- Always use `useMemo` for computed values (especially switch statements)

✅ Good:

```tsx
export const StartButton: React.FC<TStartButtonProps> = (props) => {
	const { status, onStart, onPause, className } = props;

	const { label, onClick } = React.useMemo(() => {
		switch (status) {
			case 'idle':
				return { label: 'START', onClick: onStart };
			case 'running':
				return { label: 'PAUSE', onClick: onPause };
		}
	}, [status, onStart, onPause]);

	return (
		<button onClick={onClick} className={className}>
			{label}
		</button>
	);
};

interface TStartButtonProps {
	status: 'idle' | 'running';
	onStart: () => void;
	onPause: () => void;
	className?: string;
}
```

❌ Bad:

```tsx
// Wrong: function declaration, I prefix, inline ternary
function StartButton({ status, onStart }: IStartButtonProps) {
	const label = status === 'idle' ? 'START' : 'PAUSE';
	return <button>{label}</button>;
}

interface IStartButtonProps {
	status: 'idle' | 'running';
}
```

## Props with Defaults

- Always prefer props with defaults over module-level constants
- Always destructure with default values in props

✅ Good:

```tsx
export const NumberWheel: React.FC<TNumberWheelProps> = (props) => {
	const { value, min = 1, max = 60, itemWidth = 14 } = props;
	// ...
};

interface TNumberWheelProps {
	value: number;
	min?: number;
	max?: number;
	itemWidth?: number;
}
```

❌ Bad:

```tsx
const ITEM_WIDTH = 14;  // Wrong: module-level constant

export const NumberWheel: React.FC<TNumberWheelProps> = (props) => {
	// Uses ITEM_WIDTH instead of prop
};
```

## Hooks

- Always prefix custom hooks with `use`
- Always use `React.useCallback` for event handlers passed to children
- Always use `React.useMemo` for expensive computations
- Always use `React.useRef` for mutable values that don't trigger re-renders

✅ Good:

```tsx
export function useTimer(): TUseTimerReturn {
	const [state, setState] = React.useState<TTimerState>(initialState);
	const intervalRef = React.useRef<number | null>(null);

	const start = React.useCallback(() => {
		setState((prev) => ({ ...prev, status: 'running' }));
	}, []);

	return { state, start };
}

interface TUseTimerReturn {
	state: TTimerState;
	start: () => void;
}
```

## Component Structure

Larger components follow a consistent section order (skip MARK comments for small components):

1. **Top** - Props destructuring, useState, useRef, useMemo
2. **`// MARK: - Actions`** - useCallback handlers
3. **`// MARK: - Effects`** - useEffect hooks
4. **`// MARK: - UI`** - return JSX

✅ Good:

```tsx
export const Timer: React.FC<TTimerProps> = (props) => {
	const { initialTime, onComplete } = props;

	const [time, setTime] = React.useState(initialTime);
	const intervalRef = React.useRef<number | null>(null);

	const isRunning = React.useMemo(() => time > 0, [time]);

	// MARK: - Actions

	const start = React.useCallback(() => {
		intervalRef.current = window.setInterval(() => {
			setTime((prev) => prev - 1);
		}, 1000);
	}, []);

	const stop = React.useCallback(() => {
		if (intervalRef.current != null) {
			clearInterval(intervalRef.current);
		}
	}, []);

	// MARK: - Effects

	React.useEffect(() => {
		if (time === 0) {
			stop();
			onComplete?.();
		}
	}, [time, stop, onComplete]);

	// MARK: - UI

	return (
		<div>
			{/* Time display */}
			<span>{time}</span>

			{/* Controls */}
			<button onClick={start}>Start</button>
		</div>
	);
};
```

## Comments

- Use `{/* Label */}` in JSX for layout structure (helps skimming)
- Use `// Note:` prefix for non-obvious explanations
- Never restate what code does - explain WHY if not obvious
- Keep comments concise - one line when possible

## File Structure

- Always organize features with `components/` and `hooks/` folders
- Always use barrel exports with `export * from`

✅ Good:

```
features/
  timer/
    components/
      TimerDial.tsx
      StartButton.tsx
      index.ts          # export * from './TimerDial'; etc.
    hooks/
      use-timer.ts
      index.ts          # export * from './use-timer';
    TimerView.tsx
    types.ts
    index.ts            # export * from './TimerView'; etc.
```

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
