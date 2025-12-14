# Component Patterns Reference

Detailed patterns and examples for common MUI v5 components.

## Forms and Inputs

### TextField Patterns

**Basic text input:**
```typescript
interface UserFormProps {
  value: string;
  onChange: (value: string) => void;
  error?: string;
}

const UserNameField: React.FC<UserFormProps> = ({ value, onChange, error }) => (
  <TextField
    label="Full Name"
    value={value}
    onChange={(e) => onChange(e.target.value)}
    error={!!error}
    helperText={error || 'Enter your full legal name'}
    required
    fullWidth
  />
);
```

**Multiline text:**
```typescript
<TextField
  label="Description"
  multiline
  rows={4}
  value={description}
  onChange={(e) => setDescription(e.target.value)}
  placeholder="Enter a detailed description..."
  fullWidth
/>
```

**Number input:**
```typescript
<TextField
  label="Age"
  type="number"
  value={age}
  onChange={(e) => setAge(Number(e.target.value))}
  inputProps={{ min: 0, max: 120 }}
  fullWidth
/>
```

### Select and Autocomplete

**Simple Select:**
```typescript
<FormControl fullWidth>
  <InputLabel id="role-label">Role</InputLabel>
  <Select
    labelId="role-label"
    value={role}
    label="Role"
    onChange={(e) => setRole(e.target.value)}
  >
    <MenuItem value="admin">Admin</MenuItem>
    <MenuItem value="user">User</MenuItem>
    <MenuItem value="guest">Guest</MenuItem>
  </Select>
</FormControl>
```

**Autocomplete with search:**
```typescript
<Autocomplete
  options={users}
  getOptionLabel={(user) => user.name}
  value={selectedUser}
  onChange={(_, newValue) => setSelectedUser(newValue)}
  renderInput={(params) => (
    <TextField {...params} label="Select User" />
  )}
  fullWidth
/>
```

### Checkbox and Radio

**Checkbox group:**
```typescript
<FormControl component="fieldset">
  <FormLabel component="legend">Preferences</FormLabel>
  <FormGroup>
    <FormControlLabel
      control={<Checkbox checked={emailNotifs} onChange={(e) => setEmailNotifs(e.target.checked)} />}
      label="Email notifications"
    />
    <FormControlLabel
      control={<Checkbox checked={smsNotifs} onChange={(e) => setSmsNotifs(e.target.checked)} />}
      label="SMS notifications"
    />
  </FormGroup>
</FormControl>
```

**Radio group:**
```typescript
<FormControl component="fieldset">
  <FormLabel component="legend">Account Type</FormLabel>
  <RadioGroup value={accountType} onChange={(e) => setAccountType(e.target.value)}>
    <FormControlLabel value="personal" control={<Radio />} label="Personal" />
    <FormControlLabel value="business" control={<Radio />} label="Business" />
  </RadioGroup>
</FormControl>
```

## Dialogs and Modals

### Confirmation Dialog

```typescript
interface ConfirmDialogProps {
  open: boolean;
  title: string;
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
  confirmText?: string;
  cancelText?: string;
}

const ConfirmDialog: React.FC<ConfirmDialogProps> = ({
  open,
  title,
  message,
  onConfirm,
  onCancel,
  confirmText = 'Confirm',
  cancelText = 'Cancel',
}) => (
  <Dialog open={open} onClose={onCancel}>
    <DialogTitle>{title}</DialogTitle>
    <DialogContent>
      <DialogContentText>{message}</DialogContentText>
    </DialogContent>
    <DialogActions>
      <Button onClick={onCancel}>{cancelText}</Button>
      <Button onClick={onConfirm} variant="contained" color="primary">
        {confirmText}
      </Button>
    </DialogActions>
  </Dialog>
);
```

### Form Dialog

```typescript
const FormDialog: React.FC = () => {
  const [open, setOpen] = useState(false);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = () => {
    // Process form
    setOpen(false);
  };

  return (
    <Dialog open={open} onClose={() => setOpen(false)} maxWidth="sm" fullWidth>
      <DialogTitle>Add User</DialogTitle>
      <DialogContent>
        <Stack spacing={2} sx={{ mt: 1 }}>
          <TextField
            label="Name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            fullWidth
            autoFocus
          />
          <TextField
            label="Email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            fullWidth
          />
        </Stack>
      </DialogContent>
      <DialogActions>
        <Button onClick={() => setOpen(false)}>Cancel</Button>
        <Button onClick={handleSubmit} variant="contained">
          Add
        </Button>
      </DialogActions>
    </Dialog>
  );
};
```

## Tables

### Basic Data Table

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

interface UserTableProps {
  users: User[];
  onEdit: (user: User) => void;
  onDelete: (userId: number) => void;
}

const UserTable: React.FC<UserTableProps> = ({ users, onEdit, onDelete }) => (
  <TableContainer component={Paper}>
    <Table>
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
          <TableRow key={user.id} hover>
            <TableCell>{user.name}</TableCell>
            <TableCell>{user.email}</TableCell>
            <TableCell>{user.role}</TableCell>
            <TableCell align="right">
              <IconButton onClick={() => onEdit(user)} aria-label="edit">
                <EditIcon />
              </IconButton>
              <IconButton onClick={() => onDelete(user.id)} aria-label="delete">
                <DeleteIcon />
              </IconButton>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  </TableContainer>
);
```

### Table with Pagination

```typescript
const PaginatedTable: React.FC = () => {
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(10);

  return (
    <>
      <TableContainer component={Paper}>
        <Table>
          {/* Table content */}
        </Table>
      </TableContainer>
      <TablePagination
        component="div"
        count={totalRows}
        page={page}
        onPageChange={(_, newPage) => setPage(newPage)}
        rowsPerPage={rowsPerPage}
        onRowsPerPageChange={(e) => setRowsPerPage(parseInt(e.target.value, 10))}
        rowsPerPageOptions={[5, 10, 25]}
      />
    </>
  );
};
```

## Layout Components

### Stack for Vertical Layouts

```typescript
// Vertical form with consistent spacing
<Stack spacing={3}>
  <TextField label="First Name" />
  <TextField label="Last Name" />
  <TextField label="Email" />
  <Button variant="contained">Submit</Button>
</Stack>

// Horizontal button group
<Stack direction="row" spacing={2}>
  <Button variant="outlined">Cancel</Button>
  <Button variant="contained">Save</Button>
</Stack>
```

### Grid for Responsive Layouts

```typescript
<Grid container spacing={3}>
  <Grid item xs={12} sm={6} md={4}>
    <Card>{/* Card content */}</Card>
  </Grid>
  <Grid item xs={12} sm={6} md={4}>
    <Card>{/* Card content */}</Card>
  </Grid>
  <Grid item xs={12} sm={6} md={4}>
    <Card>{/* Card content */}</Card>
  </Grid>
</Grid>
```

### Box for Custom Containers

```typescript
// Centered content
<Box
  sx={{
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    minHeight: '100vh',
  }}
>
  <Paper sx={{ p: 4, maxWidth: 400 }}>
    {/* Content */}
  </Paper>
</Box>

// Responsive padding
<Box sx={{ px: { xs: 2, sm: 3, md: 4 }, py: 3 }}>
  {/* Content */}
</Box>
```

## Cards

### Standard Card Pattern

```typescript
<Card>
  <CardHeader
    avatar={<Avatar>U</Avatar>}
    action={
      <IconButton aria-label="settings">
        <MoreVertIcon />
      </IconButton>
    }
    title="User Profile"
    subheader="Last updated: Today"
  />
  <CardMedia
    component="img"
    height="194"
    image="/profile.jpg"
    alt="Profile"
  />
  <CardContent>
    <Typography variant="body2" color="text.secondary">
      User description goes here
    </Typography>
  </CardContent>
  <CardActions>
    <Button size="small">Share</Button>
    <Button size="small">Learn More</Button>
  </CardActions>
</Card>
```

## Alerts and Snackbars

### Alert Component

```typescript
<Alert severity="error" onClose={() => setShowAlert(false)}>
  This is an error message
</Alert>

<Alert severity="warning">
  This is a warning message
</Alert>

<Alert severity="info">
  This is an info message
</Alert>

<Alert severity="success">
  This is a success message
</Alert>
```

### Snackbar Notification

```typescript
const [snackbar, setSnackbar] = useState<{ open: boolean; message: string; severity: 'success' | 'error' }>({
  open: false,
  message: '',
  severity: 'success',
});

<Snackbar
  open={snackbar.open}
  autoHideDuration={6000}
  onClose={() => setSnackbar({ ...snackbar, open: false })}
>
  <Alert severity={snackbar.severity} sx={{ width: '100%' }}>
    {snackbar.message}
  </Alert>
</Snackbar>
```

## Loading States

### Linear Progress

```typescript
{loading && <LinearProgress />}
```

### Circular Progress

```typescript
<Box sx={{ display: 'flex', justifyContent: 'center', p: 3 }}>
  <CircularProgress />
</Box>
```

### Skeleton Loaders

```typescript
<Stack spacing={1}>
  <Skeleton variant="text" sx={{ fontSize: '1rem' }} />
  <Skeleton variant="circular" width={40} height={40} />
  <Skeleton variant="rectangular" width={210} height={60} />
  <Skeleton variant="rounded" width={210} height={60} />
</Stack>
```

## Navigation

### AppBar with Menu

```typescript
<AppBar position="static">
  <Toolbar>
    <IconButton
      edge="start"
      color="inherit"
      aria-label="menu"
      sx={{ mr: 2 }}
    >
      <MenuIcon />
    </IconButton>
    <Typography variant="h6" sx={{ flexGrow: 1 }}>
      App Title
    </Typography>
    <Button color="inherit">Login</Button>
  </Toolbar>
</AppBar>
```

### Tabs Navigation

```typescript
const [tab, setTab] = useState(0);

<Box sx={{ borderBottom: 1, borderColor: 'divider' }}>
  <Tabs value={tab} onChange={(_, newValue) => setTab(newValue)}>
    <Tab label="Overview" />
    <Tab label="Details" />
    <Tab label="Settings" />
  </Tabs>
</Box>
<Box sx={{ p: 3 }}>
  {tab === 0 && <div>Overview Content</div>}
  {tab === 1 && <div>Details Content</div>}
  {tab === 2 && <div>Settings Content</div>}
</Box>
```

## Tooltips and Popovers

### Tooltip

```typescript
<Tooltip title="Delete item" arrow>
  <IconButton aria-label="delete">
    <DeleteIcon />
  </IconButton>
</Tooltip>
```

### Popover

```typescript
const [anchorEl, setAnchorEl] = useState<HTMLElement | null>(null);

<Button onClick={(e) => setAnchorEl(e.currentTarget)}>
  Open Popover
</Button>
<Popover
  open={Boolean(anchorEl)}
  anchorEl={anchorEl}
  onClose={() => setAnchorEl(null)}
  anchorOrigin={{
    vertical: 'bottom',
    horizontal: 'left',
  }}
>
  <Box sx={{ p: 2 }}>
    <Typography>Popover content</Typography>
  </Box>
</Popover>
```
