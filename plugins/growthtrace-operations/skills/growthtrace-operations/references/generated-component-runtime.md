# Generated component runtime snapshot

Offline fallback for capability contract `1.4.0`.

The adjacent `generated-component-contract.json` is the deterministically generated, exhaustive machine snapshot. This Markdown explains how to apply it; if a detail differs, use the JSON snapshot's narrower rule.

- Contract hash: `3f72f7d6e7997d4df25b99349e6387ea509451a214cd51ebf4206c805e538a11`
- Compiler: `2.2.0`
- Runtime: `1`
- Policy: `2.2.0`
- Live authority: `get_generated_component_capabilities`

Always call the live capability tool before writing TSX when it is available. Use this snapshot only when the tool is temporarily unavailable. If a live `1.4.0` contract has a different hash, stop all component writes and report a plugin distribution mismatch. This Skill supports contract `1.x`; for a newer major version, use only capabilities common to this snapshot or ask the user to upgrade.

## Source shape

- One TSX file, at most 50,000 characters and 8,000 syntax nodes.
- Exactly one `export default function`. Non-exported helper components/functions in the same file are allowed when necessary.
- Do not add other exports, app roots, `createRoot`, iframe/srcDoc, script injection, network calls, dynamic code, Node APIs, sensitive browser globals, prototype reflection, or raw DOM escape paths.
- Tailwind class names must be static. Component CSS is extracted. Do not depend on arbitrary inline style objects or remote media in the sandbox review.
- Render classification is `static` or `interactive`; Hooks, event handlers, refs, browser behavior, and interactive libraries are classified conservatively as interactive.

## Personal component dependencies

Use `list_generated_components`, then `get_generated_component` to inspect purpose, props and article variable paths. Select a published immutable version with `dependencies: [{alias, componentId, versionId}]`, then use `import Alias from '@growthtrace/components/Alias'`. A component may import up to 10 direct dependencies, 20 transitive components and 4 levels. Cycles and unavailable components are rejected. Child code, styles and variables are compiled into the parent; archiving any child revokes parent script loads while published SSR remains available. Changing a dependency version requires a new preview and owner approval.

## Allowed imports

- `react`: `Children`, `Component`, `Fragment`, `Profiler`, `PureComponent`, `StrictMode`, `Suspense`, `cloneElement`, `createContext`, `createRef`, `forwardRef`, `isValidElement`, `lazy`, `memo`, `startTransition`, `use`, `useCallback`, `useContext`, `useDebugValue`, `useDeferredValue`, `useEffect`, `useId`, `useImperativeHandle`, `useInsertionEffect`, `useLayoutEffect`, `useMemo`, `useReducer`, `useRef`, `useState`, `useSyncExternalStore`, `useTransition`. Use JSX, not `createElement`.
- `@growthtrace/component-runtime`: `useArticleVariable`, `useComponentPortalHost`.
- `@growthtrace/ui`: `Badge`, `Button`, `Card`, `CardContent`, `CardDescription`, `CardFooter`, `CardHeader`, `CardTitle`, `Tabs`, `TabsContent`, `TabsList`, `TabsTrigger`, `Tooltip`, `TooltipContent`, `TooltipProvider`, `TooltipTrigger`.
- `@growthtrace/charts`: `EChart`.
- `@growthtrace/mdx`: `Admonition`, `Columns`, `Grid`, `AnimatedBlock`, `AnimatedCounter`, `AnimatedSection`, `HeroHeader`, `ScrollProgress`, `StaggerList`, `TextReveal`, `NestedMdx`, `DisplayMdx`, `VariableReference`.
- `@growthtrace/three`: `Canvas`, `Model`, `ModelViewer`, `useGLTF`.
- `@react-three/fiber`: `Canvas`, `useFrame`, `useThree`.
- `@react-three/drei`: `OrbitControls`, `PerspectiveCamera`, `OrthographicCamera`, `Center`, `Bounds`, `Float`, `ContactShadows`, `useGLTF`, `useAnimations`.
- `three`: `Vector2`, `Vector3`, `Vector4`, `Euler`, `Quaternion`, `Matrix3`, `Matrix4`, `Color`, `MathUtils`, `Box3`, `Sphere`, `Plane`, `Ray`, `Clock`, `Group`, `Object3D`, `Mesh`, `BoxGeometry`, `SphereGeometry`, `PlaneGeometry`, `CylinderGeometry`, `TorusGeometry`, `MeshBasicMaterial`, `MeshStandardMaterial`, `MeshPhysicalMaterial`, `DoubleSide`, `FrontSide`, `BackSide`.
- Bundled utilities: `gsap` (`gsap`), `framer-motion` (`motion`, `AnimatePresence`), `lucide-react` icons, `zod` (`z`), approved `date-fns` functions/locales, and only these Lodash subpaths: `lodash-es/chunk`, `lodash-es/clamp`, `lodash-es/groupBy`, `lodash-es/maxBy`, `lodash-es/meanBy`, `lodash-es/minBy`, `lodash-es/orderBy`, `lodash-es/sortBy`, `lodash-es/sumBy`, `lodash-es/uniq`, `lodash-es/uniqBy`. Date subpath patterns are `date-fns/*` and `date-fns/locale/*`.

An empty `allowedExports` on an enabled root means installed package named exports (currently icons and date utilities), not “no exports.” Only React permits a default root import. Namespace imports are accepted only for runtime-loader modules and expose only their listed exports. A module with `rootImportAllowed: false` (currently `lodash-es`) is available only through its listed subpaths.

Do not infer an unlisted export from package documentation or another component. The live contract is authoritative when its allowed exports are narrower or broader.

## Component API essentials

- `Button` uses standard button props plus `variant`, `size`, and `asChild`; Cards use only the sections needed. Tabs require matching `value` across trigger/content. Tooltip content must not be the only way to access essential information.
- `EChart` requires `option` and `ariaLabel`; give its container a static height class such as `h-72`.
- `Admonition` requires `type` and children. `AnimatedCounter` requires `value`; `HeroHeader` and `TextReveal` require `title` and `text`. `NestedMdx`/`DisplayMdx` strings remain display data.
- Managed `Canvas` accepts `title`/`resetKey`; `Model` and `ModelViewer` require `src`. Fiber Hooks must be used under a Canvas scene. Prefer managed Three primitives when sufficient.

The live contract's `componentApis` and categorized examples are authoritative for exact summaries.

## Runtime APIs and data

`useArticleVariable(path, fallback?)` reads a safe dotted path from the published variable snapshot. Every extracted variable path must be present in `previewVariables`; use a component-level fallback only when the live contract explicitly permits the missing value. `useComponentPortalHost()` may return `null` during SSR, and portals must stay inside that host.

Props Schema is a map from prop name to a strict rule:

```json
{
    "title": { "type": "string", "required": true, "description": "Visible heading" },
    "items": { "type": "array", "required": false }
}
```

Supported types are `string`, `number`, `boolean`, `array`, and `object`. Unknown props, missing required props, invalid types, unknown rule fields, dangerous keys, and non-JSON values are rejected. An empty Schema allows no Props. Each of `propsSchema`, `previewProps`, and `previewVariables` is limited to 100 keys and 20,000 serialized JSON characters.

Minimal component:

```tsx
export default function Example() {
    return <div className="rounded-lg p-4">Example</div>;
}
```

UI, chart, and variable examples are also present in the live contract. Start from the smallest matching example and add only behavior required by the user.

The compiler produces two different artifacts: `browserBundle` uses the public article/workbench shared runtime; `sandboxBundle` is self-contained and used only by the no-network temporary review iframe. Compiled component CSS is limited to 100,000 bytes, the browser bundle to 1,250,000 bytes, and the sandbox bundle to 6,000,000 bytes. A preview session lasts at most 86,400 seconds; its fragment token lasts at most 1,800 seconds and is rotated only by the explicit refresh tool. Never request, download, edit, return, or execute either bundle yourself.
