# Project structure

```
src/
├── app/                                    # Application configuration & setup
│   ├── providers/                         # Context providers & app setup
│   │   ├── AppProvider.tsx               # Main app provider wrapper
│   │   ├── AuthProvider.tsx              # Authentication context
│   │   ├── ErrorBoundary.tsx             # Global error boundary
│   │   ├── InfrastructureProvider.tsx    # Analytics, monitoring setup
│   │   └── QueryProvider.tsx             # React Query configuration
│   ├── router/                           # Routing configuration
│   │   ├── AppRouter.tsx                 # Main router component
│   │   ├── ProtectedRoute.tsx            # Auth guards
│   │   ├── routes.tsx                    # Route definitions
│   │   └── types.ts                      # Router types
│   ├── store/                            # Global state stores
│   │   ├── authStore.ts                  # Authentication state
│   │   ├── notificationStore.ts          # Global notifications
│   │   ├── uiStore.ts                    # UI state (modals, drawer)
│   │   └── index.ts                      # Store exports
│   └── App.tsx                           # Root app component
│
├── shared/                               # Shared utilities & common code
│   ├── api/                             # API layer & configuration
│   │   ├── client.ts                    # API client setup (ConnectRPC)
│   │   ├── interceptors.ts              # Request/response interceptors
│   │   ├── types.ts                     # API types
│   │   └── endpoints.ts                 # API endpoint definitions
│   ├── lib/                             # Third-party library configurations
│   │   ├── msw/                         # Mock Service Worker setup
│   │   │   ├── handlers.ts              # MSW handlers
│   │   │   ├── worker.ts                # Service worker setup
│   │   │   └── mockData.ts              # Mock data generators
│   │   ├── react-query.ts               # React Query config
│   │   ├── i18n.ts                      # Internationalization setup
│   │   └── analytics.ts                 # Analytics configuration
│   ├── hooks/                           # Shared custom hooks
│   │   ├── useDebounce.ts               # Debouncing hook
│   │   ├── useLocalStorage.ts           # Local storage hook
│   │   ├── useErrorHandler.ts           # Error handling hook
│   │   ├── usePermissions.ts            # Permission checking hook
│   │   └── index.ts                     # Hook exports
│   ├── utils/                           # Utility functions
│   │   ├── formatters.ts                # Data formatting (money, date, phone)
│   │   ├── validators.ts                # Validation utilities
│   │   ├── helpers.ts                   # Generic helper functions
│   │   ├── constants.ts                 # Application constants
│   │   └── types.ts                     # Common TypeScript types
│   └── constants/                       # Global constants
│       ├── routes.ts                    # Route constants
│       ├── permissions.ts               # Permission constants
│       ├── api.ts                       # API constants
│       └── ui.ts                        # UI constants
│
├── entities/                            # Business entities (Domain layer)
│   ├── user/                           # User entity
│   │   ├── api/                        # User-specific API calls
│   │   │   ├── queries.ts              # User queries
│   │   │   ├── mutations.ts            # User mutations
│   │   │   └── types.ts                # User API types
│   │   ├── model/                      # User domain model
│   │   │   ├── schema.ts               # Zod schemas for user
│   │   │   ├── types.ts                # User domain types
│   │   │   └── store.ts                # User-specific state
│   │   ├── ui/                         # User-specific components
│   │   │   ├── UserCard.tsx            # User display card
│   │   │   ├── UserForm.tsx            # User form component
│   │   │   └── index.ts                # Component exports
│   │   └── lib/                        # User utilities
│   │       ├── validation.ts           # User validation logic
│   │       └── formatters.ts           # User data formatters
│   │
│   ├── data-record/                    # Data record entity
│   │   ├── api/
│   │   │   ├── queries.ts
│   │   │   ├── mutations.ts
│   │   │   └── types.ts
│   │   ├── model/
│   │   │   ├── schema.ts
│   │   │   ├── types.ts
│   │   │   └── store.ts
│   │   ├── ui/
│   │   │   ├── RecordCard.tsx
│   │   │   ├── RecordForm.tsx
│   │   │   └── index.ts
│   │   └── lib/
│   │       ├── validation.ts
│   │       └── formatters.ts
│   │
│   └── [other-entities]/              # Additional business entities
│       └── ...                        # Same structure as above
│
├── features/                          # Feature modules (Application layer)
│   ├── auth/                         # Authentication feature
│   │   ├── api/
│   │   │   ├── authQueries.ts        # Auth API calls
│   │   │   └── types.ts              # Auth API types
│   │   ├── model/
│   │   │   ├── authStore.ts          # Auth state management
│   │   │   └── types.ts              # Auth domain types
│   │   ├── ui/
│   │   │   ├── LoginForm.tsx         # Login form component
│   │   │   ├── LogoutButton.tsx      # Logout component
│   │   │   └── index.ts              # Component exports
│   │   ├── lib/
│   │   │   ├── validation.ts         # Auth validation
│   │   │   └── helpers.ts            # Auth helpers
│   │   └── hooks/
│   │       ├── useAuth.ts            # Main auth hook
│   │       ├── useLogin.ts           # Login logic hook
│   │       └── usePermissions.ts     # Permission hook
│   │
│   ├── data-management/              # Core data management feature
│   │   ├── api/
│   │   │   ├── dataQueries.ts        # Data fetching queries
│   │   │   ├── dataMutations.ts      # Data modification mutations
│   │   │   └── types.ts              # Data API types
│   │   ├── model/
│   │   │   ├── dataStore.ts          # Data state management
│   │   │   ├── filterStore.ts        # Filter state
│   │   │   └── types.ts              # Data domain types
│   │   ├── ui/
│   │   │   ├── DataTable.tsx         # Main data table
│   │   │   ├── FilterPanel.tsx       # Filtering interface
│   │   │   ├── DataActions.tsx       # Action buttons
│   │   │   └── index.ts              # Component exports
│   │   ├── lib/
│   │   │   ├── validation.ts         # Data validation
│   │   │   ├── formatters.ts         # Data formatting
│   │   │   └── helpers.ts            # Data helpers
│   │   └── hooks/
│   │       ├── useDataManagement.ts  # Main data hook
│   │       ├── useDataFilters.ts     # Filtering logic
│   │       └── useDataActions.ts     # Action handlers
│   │
│   ├── import-export/                # Import/Export feature
│   │   ├── api/
│   │   │   ├── importQueries.ts      # Import API calls
│   │   │   ├── exportQueries.ts      # Export API calls
│   │   │   └── types.ts              # Import/Export types
│   │   ├── model/
│   │   │   ├── importStore.ts        # Import state
│   │   │   └── types.ts              # Import domain types
│   │   ├── ui/
│   │   │   ├── ImportDialog.tsx      # Import modal
│   │   │   ├── ExportButton.tsx      # Export button
│   │   │   ├── ImportValidator.tsx   # Import validation
│   │   │   └── index.ts              # Component exports
│   │   ├── lib/
│   │   │   ├── csvParser.ts          # CSV parsing logic
│   │   │   ├── pdfGenerator.ts       # PDF generation
│   │   │   └── validation.ts         # Import validation
│   │   └── hooks/
│   │       ├── useImport.ts          # Import logic hook
│   │       └── useExport.ts          # Export logic hook
│   │
│   ├── forms/                        # Multi-step forms feature
│   │   ├── model/
│   │   │   ├── formStore.ts          # Form state management
│   │   │   └── types.ts              # Form types
│   │   ├── ui/
│   │   │   ├── MultiStepForm.tsx     # Main form component
│   │   │   ├── StepIndicator.tsx     # Step navigation
│   │   │   ├── FormStep.tsx          # Individual step
│   │   │   └── index.ts              # Component exports
│   │   ├── lib/
│   │   │   ├── validation.ts         # Cross-step validation
│   │   │   └── persistence.ts        # Draft saving logic
│   │   └── hooks/
│   │       ├── useMultiStepForm.ts   # Main form hook
│   │       └── useFormPersistence.ts # Draft persistence
│   │
│   ├── filtering/                    # Advanced filtering feature
│   │   ├── model/
│   │   │   ├── filterStore.ts        # Filter state
│   │   │   └── types.ts              # Filter types
│   │   ├── ui/
│   │   │   ├── FilterBuilder.tsx     # Filter construction UI
│   │   │   ├── FilterChips.tsx       # Active filter display
│   │   │   ├── SearchInput.tsx       # Debounced search
│   │   │   └── index.ts              # Component exports
│   │   ├── lib/
│   │   │   ├── filterUtils.ts        # Filter utilities
│   │   │   └── operators.ts          # Filter operators
│   │   └── hooks/
│   │       ├── useFiltering.ts       # Main filtering hook
│   │       └── useDebouncedSearch.ts # Debounced search
│   │
│   └── notifications/                # Notification system feature
│       ├── model/
│       │   ├── notificationStore.ts  # Notification state
│       │   └── types.ts              # Notification types
│       ├── ui/
│       │   ├── Toast.tsx             # Toast component
│       │   ├── NotificationCenter.tsx# Notification center
│       │   ├── Alert.tsx             # Alert component
│       │   └── index.ts              # Component exports
│       ├── lib/
│       │   └── notificationHelpers.ts# Notification utilities
│       └── hooks/
│           └── useNotifications.ts   # Notification hook
│
├── widgets/                          # Composite UI blocks (Presentation layer)
│   ├── data-table/                  # Advanced data table widget
│   │   ├── DataTable.tsx            # Main table component
│   │   ├── TableHeader.tsx          # Table header
│   │   ├── TableRow.tsx             # Table row
│   │   ├── TableCell.tsx            # Table cell
│   │   ├── TablePagination.tsx      # Pagination controls
│   │   ├── TableActions.tsx         # Row actions
│   │   ├── index.ts                 # Widget exports
│   │   └── types.ts                 # Table types
│   │
│   ├── tree-table/                  # Hierarchical table widget
│   │   ├── TreeTable.tsx            # Main tree table
│   │   ├── TreeNode.tsx             # Tree node component
│   │   ├── TreeRow.tsx              # Tree row
│   │   ├── index.ts                 # Widget exports
│   │   └── types.ts                 # Tree table types
│   │
│   ├── multi-step-form/             # Multi-step form widget
│   │   ├── MultiStepForm.tsx        # Main form widget
│   │   ├── StepIndicator.tsx        # Step navigation
│   │   ├── StepContent.tsx          # Step content area
│   │   ├── StepNavigation.tsx       # Next/Previous buttons
│   │   ├── index.ts                 # Widget exports
│   │   └── types.ts                 # Form widget types
│   │
│   ├── drawer-panels/               # Drawer panel widget
│   │   ├── Drawer.tsx               # Main drawer component
│   │   ├── DrawerHeader.tsx         # Drawer header
│   │   ├── DrawerContent.tsx        # Drawer content
│   │   ├── DrawerFooter.tsx         # Drawer footer
│   │   ├── index.ts                 # Widget exports
│   │   └── types.ts                 # Drawer types
│   │
│   └── [other-widgets]/            # Additional composite widgets
│       └── ...                      # Similar structure
│
├── components/                      # Base UI components (Design System)
│   ├── static/                     # Static presentation components
│   │   ├── Text.tsx                # Text component with variants
│   │   ├── Title.tsx               # Title/heading component
│   │   ├── Icon.tsx                # Icon component
│   │   ├── Divider.tsx             # Divider component
│   │   ├── Image.tsx               # Image component
│   │   ├── Label.tsx               # Label component
│   │   └── index.ts                # Static component exports
│   │
│   ├── layout/                     # Layout components
│   │   ├── Grid.tsx                # Grid layout component
│   │   ├── GridItem.tsx            # Grid item component
│   │   ├── Space.tsx               # Spacing component
│   │   ├── Flex.tsx                # Flex container
│   │   ├── Stack.tsx               # Stack layout (H/V)
│   │   ├── Container.tsx           # Page container
│   │   ├── Section.tsx             # Content section
│   │   ├── Card.tsx                # Card container
│   │   └── index.ts                # Layout component exports
│   │
│   ├── form/                       # Form-related components
│   │   ├── Input.tsx               # Input field
│   │   ├── Textarea.tsx            # Textarea field
│   │   ├── Select.tsx              # Select dropdown
│   │   ├── Checkbox.tsx            # Checkbox component
│   │   ├── Radio.tsx               # Radio button
│   │   ├── Switch.tsx              # Toggle switch
│   │   ├── FormField.tsx           # Form field wrapper
│   │   ├── FormError.tsx           # Error display
│   │   └── index.ts                # Form component exports
│   │
│   ├── interactive/                # Interactive components
│   │   ├── Button.tsx              # Button component
│   │   ├── Link.tsx                # Link component
│   │   ├── Modal.tsx               # Modal dialog
│   │   ├── Tooltip.tsx             # Tooltip component
│   │   ├── Dropdown.tsx            # Dropdown menu
│   │   ├── Tabs.tsx                # Tab navigation
│   │   └── index.ts                # Interactive component exports
│   │
│   ├── feedback/                   # Feedback components
│   │   ├── Loading.tsx             # Loading spinner
│   │   ├── Skeleton.tsx            # Skeleton loader
│   │   ├── ProgressBar.tsx         # Progress indicator
│   │   ├── EmptyState.tsx          # Empty state display
│   │   ├── ErrorMessage.tsx        # Error message
│   │   └── index.ts                # Feedback component exports
│   │
│   └── index.ts                    # All component exports
│
├── pages/                          # Page components (Route components)
│   ├── auth/                       # Authentication pages
│   │   ├── LoginPage.tsx           # Login page
│   │   ├── ForgotPasswordPage.tsx  # Password reset page
│   │   └── index.ts                # Auth page exports
│   │
│   ├── dashboard/                  # Dashboard pages
│   │   ├── DashboardPage.tsx       # Main dashboard
│   │   ├── AnalyticsPage.tsx       # Analytics dashboard
│   │   └── index.ts                # Dashboard page exports
│   │
│   ├── data-management/            # Data management pages
│   │   ├── DataListPage.tsx        # Data listing page
│   │   ├── DataDetailPage.tsx      # Data detail page
│   │   ├── DataCreatePage.tsx      # Data creation page
│   │   ├── DataEditPage.tsx        # Data editing page
│   │   └── index.ts                # Data page exports
│   │
│   ├── settings/                   # Settings pages
│   │   ├── ProfilePage.tsx         # User profile
│   │   ├── PreferencesPage.tsx     # User preferences
│   │   ├── SystemSettingsPage.tsx  # System settings
│   │   └── index.ts                # Settings page exports
│   │
│   ├── static/                     # Static pages
│   │   ├── AboutPage.tsx           # About page
│   │   ├── HelpPage.tsx            # Help documentation
│   │   ├── NotFoundPage.tsx        # 404 page
│   │   └── index.ts                # Static page exports
│   │
│   └── index.ts                    # All page exports
│
├── layouts/                        # Layout components
│   ├── AppLayout.tsx               # Main application layout
│   ├── AuthLayout.tsx              # Authentication layout
│   ├── DashboardLayout.tsx         # Dashboard layout
│   ├── components/                 # Layout-specific components
│   │   ├── Header.tsx              # Application header
│   │   ├── Sidebar.tsx             # Navigation sidebar
│   │   ├── Footer.tsx              # Application footer
│   │   ├── Breadcrumb.tsx          # Breadcrumb navigation
│   │   └── index.ts                # Layout component exports
│   └── index.ts                    # Layout exports
│
├── styles/                         # Styling files
│   ├── globals.css                 # Global styles
│   ├── components.css              # Component-specific styles
│   ├── themes/                     # Theme configurations
│   │   ├── default.css             # Default theme
│   │   ├── dark.css                # Dark theme
│   │   └── tokens.ts               # Design tokens
│   └── utilities.css               # Utility classes
│
├── assets/                         # Static assets
│   ├── images/                     # Image files
│   │   ├── icons/                  # SVG icons
│   │   ├── logos/                  # Brand logos
│   │   └── illustrations/          # Illustration assets
│   ├── fonts/                      # Font files
│   └── locales/                    # Internationalization files
│       ├── en.json                 # English translations
│       ├── vi.json                 # Vietnamese translations
│       └── index.ts                # Locale exports
│
├── types/                          # Global TypeScript definitions
│   ├── api.ts                      # API-related types
│   ├── auth.ts                     # Authentication types
│   ├── common.ts                   # Common utility types
│   ├── entities.ts                 # Business entity types
│   ├── ui.ts                       # UI component types
│   └── index.ts                    # Type exports
│
├── config/                         # Configuration files
│   ├── env.ts                      # Environment configuration
│   ├── api.ts                      # API configuration
│   ├── router.ts                   # Router configuration
│   ├── msw.ts                      # MSW configuration
│   └── analytics.ts                # Analytics configuration
│
├── tests/                          # Test files and utilities
│   ├── __mocks__/                  # Mock files
│   ├── fixtures/                   # Test fixtures
│   ├── utils/                      # Test utilities
│   │   ├── render.tsx              # Custom render function
│   │   ├── setup.ts                # Test setup
│   │   └── mockData.ts             # Test data generators
│   └── e2e/                        # End-to-end tests
│       ├── auth.spec.ts            # Auth flow tests
│       ├── data-management.spec.ts # Data management tests
│       └── navigation.spec.ts      # Navigation tests
│
├── main.tsx                        # Application entry point
├── vite-env.d.ts                   # Vite type definitions
└── index.html                      # HTML template
```

## Key Structure Principles
## 1. Feature-Sliced Design (FSD)
entities/: Domain business logic
features/: Application features with business logic
widgets/: Composite UI blocks
components/: Pure UI components
pages/: Route-level components
## 2. Consistent Module Structure
Each feature/entity follows the same internal structure:

```
feature/
├── api/          # External API interactions
├── model/        # State management & domain logic
├── ui/           # Feature-specific UI components
├── lib/          # Feature utilities & helpers
└── hooks/        # Custom hooks for the feature
```

## 3. Separation of Concerns
Static components: Pure presentation, no business logic
Layout components: Structure and arrangement
Database Connectable: Data-driven components with business logic
Widgets: Composite components combining multiple base components
## 4. Scalability Features
Lazy loading: Pages and features can be code-split
Modular imports: Each module has index.ts for clean imports
Type safety: Comprehensive TypeScript coverage
Testing: Dedicated test structure with utilities
## 5. Developer Experience
Clear boundaries: Each layer has specific responsibilities
Consistent patterns: Same structure across all modules
Easy navigation: Logical folder hierarchy
Maintainable: Loose coupling between modules