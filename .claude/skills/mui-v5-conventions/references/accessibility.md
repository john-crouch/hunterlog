# Accessibility Guidelines

WCAG compliance and MUI-specific accessibility patterns for creating inclusive user interfaces.

## Core Principles (WCAG 2.1)

### 1. Perceivable

Content must be presentable to users in ways they can perceive.

- Provide text alternatives for non-text content
- Provide captions and alternatives for multimedia
- Create content that can be presented in different ways without losing meaning
- Make it easier for users to see and hear content

### 2. Operable

User interface components and navigation must be operable.

- Make all functionality available from a keyboard
- Give users enough time to read and use content
- Do not use content that causes seizures or physical reactions
- Help users navigate and find content

### 3. Understandable

Information and operation of the user interface must be understandable.

- Make text readable and understandable
- Make content appear and operate in predictable ways
- Help users avoid and correct mistakes

### 4. Robust

Content must be robust enough to be interpreted by a wide variety of user agents.

- Maximize compatibility with current and future user tools
- Ensure proper HTML semantics
- Provide clear labels and instructions

## MUI-Specific Accessibility Patterns

### Forms and Inputs

**Always provide labels:**
```typescript
// Good
<TextField label="Email Address" id="email" />

// Bad - No label
<TextField placeholder="Email" />
```

**Associate labels with inputs:**
```typescript
<FormControl>
  <FormLabel id="gender-label">Gender</FormLabel>
  <RadioGroup aria-labelledby="gender-label">
    <FormControlLabel value="female" control={<Radio />} label="Female" />
    <FormControlLabel value="male" control={<Radio />} label="Male" />
  </RadioGroup>
</FormControl>
```

**Provide helpful error messages:**
```typescript
<TextField
  label="Email"
  error={!!emailError}
  helperText={emailError || 'We will never share your email'}
  aria-describedby="email-helper-text"
/>
```

**Required field indication:**
```typescript
<TextField
  label="Full Name"
  required
  aria-required="true"
/>
```

### Buttons and Interactive Elements

**Icon buttons need labels:**
```typescript
// Good
<IconButton aria-label="delete item" onClick={handleDelete}>
  <DeleteIcon />
</IconButton>

// Bad - No label
<IconButton onClick={handleDelete}>
  <DeleteIcon />
</IconButton>
```

**Button groups need clear labels:**
```typescript
<ButtonGroup aria-label="text alignment">
  <Button aria-label="left align"><AlignLeftIcon /></Button>
  <Button aria-label="center align"><AlignCenterIcon /></Button>
  <Button aria-label="right align"><AlignRightIcon /></Button>
</ButtonGroup>
```

**Loading buttons:**
```typescript
<LoadingButton
  loading={isLoading}
  loadingPosition="start"
  startIcon={<SaveIcon />}
  aria-busy={isLoading}
  aria-label={isLoading ? 'Saving...' : 'Save changes'}
>
  Save
</LoadingButton>
```

### Dialogs and Modals

**Proper dialog structure:**
```typescript
<Dialog
  open={open}
  onClose={handleClose}
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
>
  <DialogTitle id="dialog-title">
    Confirm Deletion
  </DialogTitle>
  <DialogContent>
    <DialogContentText id="dialog-description">
      Are you sure you want to delete this item? This action cannot be undone.
    </DialogContentText>
  </DialogContent>
  <DialogActions>
    <Button onClick={handleClose}>Cancel</Button>
    <Button onClick={handleConfirm} color="error" autoFocus>
      Delete
    </Button>
  </DialogActions>
</Dialog>
```

**Focus management:**
```typescript
const DialogComponent = () => {
  const closeButtonRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (open) {
      // Focus first interactive element
      closeButtonRef.current?.focus();
    }
  }, [open]);

  return (
    <Dialog open={open} onClose={handleClose}>
      {/* Dialog content */}
    </Dialog>
  );
};
```

### Tables

**Accessible table structure:**
```typescript
<Table aria-label="user list">
  <TableHead>
    <TableRow>
      <TableCell>Name</TableCell>
      <TableCell>Email</TableCell>
      <TableCell>Role</TableCell>
      <TableCell align="right">Actions</TableCell>
    </TableRow>
  </TableHead>
  <TableBody>
    {users.map((user) => (
      <TableRow key={user.id}>
        <TableCell>{user.name}</TableCell>
        <TableCell>{user.email}</TableCell>
        <TableCell>{user.role}</TableCell>
        <TableCell align="right">
          <IconButton
            aria-label={`edit ${user.name}`}
            onClick={() => handleEdit(user.id)}
          >
            <EditIcon />
          </IconButton>
          <IconButton
            aria-label={`delete ${user.name}`}
            onClick={() => handleDelete(user.id)}
          >
            <DeleteIcon />
          </IconButton>
        </TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
```

**Sortable columns:**
```typescript
<TableCell
  sortDirection={orderBy === 'name' ? order : false}
>
  <TableSortLabel
    active={orderBy === 'name'}
    direction={orderBy === 'name' ? order : 'asc'}
    onClick={() => handleSort('name')}
    aria-label="sort by name"
  >
    Name
  </TableSortLabel>
</TableCell>
```

### Navigation

**AppBar with proper roles:**
```typescript
<AppBar position="static">
  <Toolbar>
    <IconButton
      edge="start"
      color="inherit"
      aria-label="open navigation menu"
      onClick={handleMenuOpen}
    >
      <MenuIcon />
    </IconButton>
    <Typography variant="h6" component="h1" sx={{ flexGrow: 1 }}>
      Application Title
    </Typography>
    <nav aria-label="main navigation">
      <Button color="inherit">Home</Button>
      <Button color="inherit">About</Button>
      <Button color="inherit">Contact</Button>
    </nav>
  </Toolbar>
</AppBar>
```

**Breadcrumbs navigation:**
```typescript
<Breadcrumbs aria-label="breadcrumb navigation">
  <Link href="/" color="inherit" aria-label="go to home">
    Home
  </Link>
  <Link href="/products" color="inherit" aria-label="go to products">
    Products
  </Link>
  <Typography color="text.primary" aria-current="page">
    Product Details
  </Typography>
</Breadcrumbs>
```

**Tabs with keyboard navigation:**
```typescript
<Tabs
  value={value}
  onChange={handleChange}
  aria-label="navigation tabs"
>
  <Tab label="Overview" id="tab-0" aria-controls="tabpanel-0" />
  <Tab label="Details" id="tab-1" aria-controls="tabpanel-1" />
  <Tab label="Settings" id="tab-2" aria-controls="tabpanel-2" />
</Tabs>
<TabPanel value={value} index={0} id="tabpanel-0" aria-labelledby="tab-0">
  Overview content
</TabPanel>
```

### Alerts and Notifications

**Accessible alerts:**
```typescript
<Alert severity="error" role="alert" aria-live="assertive">
  An error occurred while saving your changes
</Alert>

<Alert severity="info" role="status" aria-live="polite">
  Your profile has been updated
</Alert>
```

**Snackbar notifications:**
```typescript
<Snackbar
  open={open}
  autoHideDuration={6000}
  onClose={handleClose}
  aria-live="polite"
  aria-atomic="true"
>
  <Alert severity="success" onClose={handleClose}>
    Operation completed successfully
  </Alert>
</Snackbar>
```

### Loading States

**Accessible loading indicators:**
```typescript
// For page-level loading
<Box
  sx={{ display: 'flex', justifyContent: 'center', p: 3 }}
  role="status"
  aria-label="Loading content"
>
  <CircularProgress />
  <Typography sx={{ ml: 2 }}>Loading...</Typography>
</Box>

// For inline loading
{isLoading ? (
  <CircularProgress size={20} aria-label="loading" />
) : (
  <Typography>Content</Typography>
)}
```

**Skeleton screens:**
```typescript
<Box aria-busy={isLoading} aria-live="polite">
  {isLoading ? (
    <Stack spacing={1}>
      <Skeleton variant="text" width="60%" />
      <Skeleton variant="rectangular" height={118} />
      <Skeleton variant="text" width="40%" />
    </Stack>
  ) : (
    <ActualContent />
  )}
</Box>
```

## Color and Contrast

### Ensure Sufficient Contrast

**Minimum contrast ratios (WCAG AA):**
- Normal text: 4.5:1
- Large text (18pt+ or 14pt+ bold): 3:1
- UI components and graphics: 3:1

**Use theme palette for consistent contrast:**
```typescript
// Good - Uses theme colors with guaranteed contrast
<Button variant="contained" color="primary">
  Primary Action
</Button>

// Bad - Custom colors without contrast verification
<Button sx={{ bgcolor: '#ffcccc', color: '#ff0000' }}>
  Poor Contrast
</Button>
```

**Text on backgrounds:**
```typescript
// Ensure readable contrast
<Box sx={{ bgcolor: 'background.paper', color: 'text.primary' }}>
  <Typography>Readable text</Typography>
</Box>

// For custom backgrounds, check contrast
<Box sx={{ bgcolor: 'primary.main', color: 'primary.contrastText' }}>
  <Typography>Text on primary background</Typography>
</Box>
```

## Focus Management

### Visible Focus Indicators

**Never remove focus outlines without replacement:**
```typescript
// Bad - Removes focus indicator
<Button sx={{ '&:focus': { outline: 'none' } }}>
  No Focus
</Button>

// Good - Custom focus indicator
<Button
  sx={{
    '&:focus-visible': {
      outline: '2px solid',
      outlineColor: 'primary.main',
      outlineOffset: 2,
    },
  }}
>
  Custom Focus
</Button>
```

**Skip to content link:**
```typescript
<Link
  href="#main-content"
  sx={{
    position: 'absolute',
    left: '-9999px',
    '&:focus': {
      position: 'static',
      left: 'auto',
    },
  }}
>
  Skip to main content
</Link>
```

### Keyboard Navigation

**Ensure keyboard operability:**
```typescript
// Good - Proper button
<Button onClick={handleClick}>Action</Button>

// Bad - Div with onClick
<div onClick={handleClick}>Action</div>

// Acceptable - Div with proper ARIA and keyboard handler
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  Action
</div>
```

**Menu keyboard navigation:**
```typescript
<Menu
  open={open}
  onClose={handleClose}
  MenuListProps={{
    'aria-labelledby': 'menu-button',
    role: 'menu',
  }}
>
  <MenuItem onClick={handleOption1} role="menuitem">
    Option 1
  </MenuItem>
  <MenuItem onClick={handleOption2} role="menuitem">
    Option 2
  </MenuItem>
</Menu>
```

## Screen Reader Support

### Meaningful Labels

**Provide context for screen readers:**
```typescript
// Good - Clear label
<IconButton aria-label="add new item">
  <AddIcon />
</IconButton>

// Better - Contextual label
<IconButton aria-label="add new user to team">
  <AddIcon />
</IconButton>
```

**Use aria-describedby for additional context:**
```typescript
<TextField
  label="Password"
  type="password"
  aria-describedby="password-requirements"
/>
<FormHelperText id="password-requirements">
  Must be at least 8 characters with one uppercase letter
</FormHelperText>
```

### Live Regions

**Announce dynamic content changes:**
```typescript
<Box aria-live="polite" aria-atomic="true">
  {`${items.length} items in cart`}
</Box>

// For urgent updates
<Box aria-live="assertive" role="alert">
  {errorMessage}
</Box>
```

## Common Accessibility Checklist

- [ ] All images have alt text
- [ ] All form inputs have labels
- [ ] All icon buttons have aria-label
- [ ] Color is not the only means of conveying information
- [ ] Contrast ratios meet WCAG AA standards (4.5:1 for normal text)
- [ ] All interactive elements are keyboard accessible
- [ ] Focus indicators are visible
- [ ] Form validation errors are announced to screen readers
- [ ] Dynamic content changes are announced via aria-live
- [ ] Dialogs properly manage focus
- [ ] Skip navigation link is available for keyboard users
- [ ] Page has meaningful title
- [ ] Headings are used in logical order (h1 → h2 → h3)
- [ ] Links have descriptive text (avoid "click here")

## Testing Tools

### Browser Extensions
- axe DevTools (Chrome/Firefox)
- WAVE (Chrome/Firefox)
- Lighthouse (Chrome DevTools)

### Screen Readers
- NVDA (Windows, free)
- JAWS (Windows, commercial)
- VoiceOver (macOS/iOS, built-in)

### Keyboard Testing
- Tab through all interactive elements
- Test common shortcuts (Enter, Space, Escape, Arrow keys)
- Ensure focus is visible and logical

### Automated Testing
```typescript
import { axe } from 'jest-axe';

test('component has no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```
