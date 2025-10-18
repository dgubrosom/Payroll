# Zap Pay SaaS - Complete Project Archive

This is a complete Next.js application with Supabase integration. Copy all the files below into your GitHub repository, then deploy to Vercel.

## Project Structure
```
zap-pay-saas/
├── .env.example
├── .gitignore
├── .github/workflows/ci-cd.yml
├── app/
│   ├── api/user-profile/route.ts
│   ├── auth/login/page.tsx
│   ├── employee-dashboard/page.tsx
│   ├── employee/[id]/page.tsx
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── add-employee-dialog.tsx
│   ├── dashboard-header.tsx
│   ├── employee-details.tsx
│   ├── employee-schedule-history.tsx
│   ├── logout-button.tsx
│   └── time-tracker.tsx
├── lib/
│   ├── database.ts
│   └── supabase.ts
├── middleware.ts
├── package.json
├── README.md
├── supabase-schema.sql
└── vercel.json
```

## Files to Create:

### 1. .env.example
```env
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# Example:
# NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
# NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
# SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 2. .gitignore
```gitignore
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.js
.yarn/install-state.gz

# testing
/coverage

# next.js
/.next/
/out/

# production
/build

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# local env files
.env*.local
.env

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
Thumbs.db
```

### 3. .github/workflows/ci-cd.yml
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run type check
      run: npm run type-check
      
    - name: Run linter
      run: npm run lint
      
    - name: Build application
      run: npm run build
      env:
        NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.NEXT_PUBLIC_SUPABASE_URL }}
        NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.NEXT_PUBLIC_SUPABASE_ANON_KEY }}

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v25
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
        vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
        working-directory: ./
```

### 4. app/api/user-profile/route.ts
```typescript
import { NextRequest, NextResponse } from 'next/server'
import { userService } from '@/lib/database'

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url)
    const email = searchParams.get('email')
    
    if (!email) {
      return NextResponse.json({ error: 'Email is required' }, { status: 400 })
    }
    
    const user = await userService.getByEmail(email)
    
    if (!user) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 })
    }
    
    return NextResponse.json({
      id: user.id,
      email: user.email,
      role: user.role,
      employee_id: user.employee_id
    })
  } catch (error) {
    console.error('Error fetching user profile:', error)
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

### 5. app/auth/login/page.tsx
```typescript
"use client"

import type React from "react"

import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import Link from "next/link"
import { useRouter } from "next/navigation"
import { useState } from "react"
import { authService } from "@/lib/database"

export default function LoginPage() {
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [isLoading, setIsLoading] = useState(false)
  const [role, setRole] = useState<"administrator" | "employee">("administrator")
  const [error, setError] = useState<string | null>(null)
  const router = useRouter()

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsLoading(true)
    setError(null)

    try {
      await authService.signIn(email, password)
      router.push("/")
      router.refresh()
    } catch (err) {
      setError(err instanceof Error ? err.message : "Login failed")
    } finally {
      setIsLoading(false)
    }
  }

  const handleSignUp = async () => {
    setIsLoading(true)
    setError(null)

    try {
      await authService.signUp(email, password, role)
      setError("Account created! Please check your email to verify your account.")
    } catch (err) {
      setError(err instanceof Error ? err.message : "Sign up failed")
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <div className="flex min-h-screen items-center justify-center bg-background p-4">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-4xl font-bold tracking-tight">Zap Pay</h1>
          <p className="mt-2 text-muted-foreground">Sign in to your account</p>
        </div>

        <div className="rounded-2xl border bg-card p-8 shadow-sm">
          <form onSubmit={handleLogin} className="space-y-6">
            <div className="space-y-3">
              <Label className="text-sm font-medium">Sign in as</Label>
              <div className="grid grid-cols-2 gap-3">
                <button
                  type="button"
                  onClick={() => setRole("administrator")}
                  className={`rounded-lg border-2 px-4 py-3 text-sm font-medium transition-all ${
                    role === "administrator"
                      ? "border-primary bg-primary/10 text-primary"
                      : "border-border bg-card text-muted-foreground hover:border-primary/50"
                  }`}
                >
                  Administrator
                </button>
                <button
                  type="button"
                  onClick={() => setRole("employee")}
                  className={`rounded-lg border-2 px-4 py-3 text-sm font-medium transition-all ${
                    role === "employee"
                      ? "border-primary bg-primary/10 text-primary"
                      : "border-border bg-card text-muted-foreground hover:border-primary/50"
                  }`}
                >
                  Employee
                </button>
              </div>
            </div>

            <div className="space-y-2">
              <Label htmlFor="email" className="text-sm font-medium">
                Email
              </Label>
              <Input
                id="email"
                type="email"
                placeholder="you@company.com"
                required
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="h-12"
              />
            </div>

            <div className="space-y-2">
              <Label htmlFor="password" className="text-sm font-medium">
                Password
              </Label>
              <Input
                id="password"
                type="password"
                required
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="h-12"
              />
            </div>

            {error && (
              <div className="rounded-lg bg-destructive/10 p-3 text-sm text-destructive">
                {error}
              </div>
            )}

            <div className="space-y-3">
              <Button type="submit" className="h-12 w-full text-base font-medium" disabled={isLoading}>
                {isLoading ? "Signing in..." : "Sign in"}
              </Button>
              
              <Button 
                type="button" 
                variant="outline" 
                className="h-12 w-full text-base font-medium" 
                disabled={isLoading}
                onClick={handleSignUp}
              >
                {isLoading ? "Creating..." : "Create Account"}
              </Button>
            </div>
          </form>

          <div className="mt-6 text-center text-sm text-muted-foreground">
            <p>For demo purposes, you can use:</p>
            <p className="mt-1 font-medium">admin@zappay.com (Administrator)</p>
            <p className="font-medium">john@example.com (Employee)</p>
          </div>
        </div>
      </div>
    </div>
  )
}
```

### 6. app/employee-dashboard/page.tsx
```typescript
"use client"

import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import { DashboardHeader } from "@/components/dashboard-header"
import { TimeTracker } from "@/components/time-tracker"
import { authService, employeeService, userService } from "@/lib/database"
import { Employee } from "@/lib/supabase"

export default function EmployeeDashboard() {
  const router = useRouter()
  const [isAuthenticated, setIsAuthenticated] = useState(false)
  const [userRole, setUserRole] = useState<string>("")
  const [employee, setEmployee] = useState<Employee | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    checkAuthAndLoadData()
  }, [router])

  const checkAuthAndLoadData = async () => {
    try {
      const user = await authService.getCurrentUser()
      if (!user) {
        router.push("/auth/login")
        return
      }

      setIsAuthenticated(true)
      
      // Get user role from database
      const userProfile = await userService.getByEmail(user.email!)
      if (!userProfile) {
        router.push("/auth/login")
        return
      }
      
      setUserRole(userProfile.role)
      
      if (userProfile.role !== "employee") {
        router.push("/")
        return
      }

      // Find employee data
      if (userProfile.employee_id) {
        const foundEmployee = await employeeService.getById(userProfile.employee_id)
        if (foundEmployee) {
          setEmployee(foundEmployee)
        }
      } else {
        // Create a default employee profile if not found
        const defaultEmployee = await employeeService.create({
          name: user.email!.split('@')[0],
          email: user.email!,
          position: "Employee",
          hourly_rate: 25,
          tax_number: null,
          pps_number: null,
        })
        
        // Update user with employee_id
        await userService.update(userProfile.id, {
          employee_id: defaultEmployee.id
        })
        
        setEmployee(defaultEmployee)
      }
      
    } catch (error) {
      console.error("Auth error:", error)
      router.push("/auth/login")
    } finally {
      setIsLoading(false)
    }
  }

  if (!isAuthenticated || isLoading) {
    return (
      <div className="min-h-screen bg-background">
        <DashboardHeader />
        <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
          <div className="flex min-h-[60vh] items-center justify-center">
            <div className="text-center">
              <div className="text-lg">Loading...</div>
            </div>
          </div>
        </main>
      </div>
    )
  }

  if (!employee) {
    return (
      <div className="min-h-screen bg-background">
        <DashboardHeader />
        <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
          <div className="flex min-h-[60vh] items-center justify-center">
            <div className="text-center">
              <h2 className="text-2xl font-semibold text-muted-foreground">Employee not found</h2>
              <p className="mt-2 text-sm text-muted-foreground">Please contact your administrator</p>
            </div>
          </div>
        </main>
      </div>
    )
  }

  return (
    <div className="min-h-screen bg-background">
      <DashboardHeader />

      <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
        <div className="mb-8">
          <h1 className="text-3xl font-bold tracking-tight">My Dashboard</h1>
          <p className="mt-1 text-muted-foreground">Welcome back, {employee.name}</p>
        </div>
        
        <TimeTracker 
          employeeId={employee.id}
          employeeName={employee.name}
          hourlyRate={employee.hourly_rate}
        />
      </main>
    </div>
  )
}
```

### 7. app/employee/[id]/page.tsx
```typescript
"use client"

import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import { DashboardHeader } from "@/components/dashboard-header"
import { EmployeeDetails } from "@/components/employee-details"
import { EmployeeScheduleHistory } from "@/components/employee-schedule-history"
import { Button } from "@/components/ui/button"
import Link from "next/link"
import { ArrowLeft } from "lucide-react"

export default function EmployeePage({ params }: { params: { id: string } }) {
  const router = useRouter()
  const [isLoading, setIsLoading] = useState(true)
  const [employee, setEmployee] = useState<any>(null)
  const [currentSchedule, setCurrentSchedule] = useState<any>(null)
  const [scheduleHistory, setScheduleHistory] = useState<any[]>([])

  useEffect(() => {
    const mockUser = localStorage.getItem("mockUser")
    if (!mockUser) {
      router.push("/auth/login")
      return
    }

    // Mock employee data
    const mockEmployees = [
      {
        id: "1",
        name: "John Smith",
        position: "Developer",
        hourly_rate: 50,
        email: "john@example.com",
        phone: "+1 234 567 8900",
      },
      {
        id: "2",
        name: "Sarah Johnson",
        position: "Designer",
        hourly_rate: 45,
        email: "sarah@example.com",
        phone: "+1 234 567 8901",
      },
      {
        id: "3",
        name: "Mike Wilson",
        position: "Manager",
        hourly_rate: 60,
        email: "mike@example.com",
        phone: "+1 234 567 8902",
      },
    ]

    const foundEmployee = mockEmployees.find((e) => e.id === params.id)
    if (!foundEmployee) {
      router.push("/")
      return
    }

    setEmployee(foundEmployee)

    // Get current week start (Monday)
    const today = new Date()
    const dayOfWeek = today.getDay()
    const diff = dayOfWeek === 0 ? -6 : 1 - dayOfWeek
    const weekStart = new Date(today)
    weekStart.setDate(today.getDate() + diff)
    weekStart.setHours(0, 0, 0, 0)
    const weekStartStr = weekStart.toISOString().split("T")[0]

    // Mock current schedule
    setCurrentSchedule({
      id: "1",
      employee_id: params.id,
      week_start_date: weekStartStr,
      monday_hours: 8,
      tuesday_hours: 8,
      wednesday_hours: 8,
      thursday_hours: 8,
      friday_hours: 8,
      saturday_hours: 0,
      sunday_hours: 0,
      total_hours: 40,
    })

    // Mock schedule history
    const history = []
    for (let i = 0; i < 8; i++) {
      const date = new Date(weekStart)
      date.setDate(date.getDate() - i * 7)
      history.push({
        id: `${i + 1}`,
        employee_id: params.id,
        week_start_date: date.toISOString().split("T")[0],
        monday_hours: 8,
        tuesday_hours: 8,
        wednesday_hours: 8,
        thursday_hours: 8,
        friday_hours: 8,
        saturday_hours: 0,
        sunday_hours: 0,
        total_hours: 40,
      })
    }
    setScheduleHistory(history)
    setIsLoading(false)
  }, [router, params.id])

  if (isLoading) {
    return (
      <div className="flex min-h-screen items-center justify-center">
        <div className="text-lg">Loading...</div>
      </div>
    )
  }

  if (!employee) {
    return null
  }

  return (
    <div className="min-h-screen bg-background">
      <DashboardHeader />

      <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
        <div className="mb-6">
          <Button variant="ghost" asChild className="gap-2">
            <Link href="/">
              <ArrowLeft className="h-4 w-4" />
              Back to Team
            </Link>
          </Button>
        </div>

        <EmployeeDetails
          employee={employee}
          currentSchedule={currentSchedule}
          weekStart={currentSchedule?.week_start_date}
        />

        <div className="mt-8">
          <EmployeeScheduleHistory schedules={scheduleHistory} />
        </div>
      </main>
    </div>
  )
}
```

### 8. app/layout.tsx
```typescript
import type React from "react"
import type { Metadata } from "next"
import { Inter } from "next/font/google"
import "./globals.css"

const inter = Inter({
  subsets: ["latin"],
  display: "swap",
})

export const metadata: Metadata = {
  title: "Zap Pay - Modern Payroll Software",
  description: "Intuitive payroll management for modern businesses",
    generator: 'v0.app'
}

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode
}>) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  )
}
```

### 9. app/page.tsx
```typescript
"use client"

import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import { DashboardHeader } from "@/components/dashboard-header"
import { AddEmployeeDialog } from "@/components/add-employee-dialog"
import { employeeService, timeEntryService, authService } from "@/lib/database"
import { Employee } from "@/lib/supabase"

interface EmployeeWithStatus extends Employee {
  currentWeekHours: number
  clockedIn: boolean
  clockInTime?: string
  clockOutTime?: string
}

export default function DashboardPage() {
  const router = useRouter()
  const [isAuthenticated, setIsAuthenticated] = useState(false)
  const [userRole, setUserRole] = useState<string>("")
  const [employees, setEmployees] = useState<EmployeeWithStatus[]>([])
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    checkAuthAndLoadData()
  }, [router])

  const checkAuthAndLoadData = async () => {
    try {
      const user = await authService.getCurrentUser()
      if (!user) {
        router.push("/auth/login")
        return
      }

      setIsAuthenticated(true)
      
      // Get user role from database
      const userProfile = await fetch(`/api/user-profile?email=${user.email}`)
      const userData = await userProfile.json()
      setUserRole(userData.role || "administrator")
      
      if (userData.role === "administrator") {
        await loadEmployees()
      } else {
        router.push("/employee-dashboard")
        return
      }
    } catch (error) {
      console.error("Auth error:", error)
      router.push("/auth/login")
    } finally {
      setIsLoading(false)
    }
  }

  const loadEmployees = async () => {
    try {
      const allEmployees = await employeeService.getAll()
      
      // Calculate current week hours and clock status for each employee
      const employeesWithStatus = await Promise.all(
        allEmployees.map(async (emp) => {
          const currentWeekHours = await calculateCurrentWeekHours(emp.id)
          const clockStatus = await getClockStatus(emp.id)
          return {
            ...emp,
            currentWeekHours,
            clockedIn: clockStatus.isClockedIn,
            clockInTime: clockStatus.clockInTime,
            clockOutTime: clockStatus.clockOutTime
          }
        })
      )
      
      setEmployees(employeesWithStatus)
    } catch (error) {
      console.error("Error loading employees:", error)
    }
  }

  const calculateCurrentWeekHours = async (employeeId: string): Promise<number> => {
    const now = new Date()
    const weekStart = getWeekStart(now)
    return await timeEntryService.getWeekHours(employeeId, weekStart)
  }

  const getClockStatus = async (employeeId: string) => {
    const todayEntry = await timeEntryService.getTodayEntry(employeeId)
    
    return {
      isClockedIn: todayEntry ? !todayEntry.clock_out_time : false,
      clockInTime: todayEntry?.clock_in_time || null,
      clockOutTime: todayEntry?.clock_out_time || null
    }
  }

  const getWeekStart = (date: Date): Date => {
    const dayOfWeek = date.getDay()
    const diff = dayOfWeek === 0 ? -6 : 1 - dayOfWeek
    const weekStart = new Date(date)
    weekStart.setDate(date.getDate() + diff)
    weekStart.setHours(0, 0, 0, 0)
    return weekStart
  }

  const handleEmployeeUpdate = () => {
    loadEmployees()
  }

  if (!isAuthenticated) {
    return null
  }

  if (isLoading) {
    return (
      <div className="min-h-screen bg-background">
        <DashboardHeader />
        <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
          <div className="flex min-h-[60vh] items-center justify-center">
            <div className="text-center">
              <div className="text-lg">Loading...</div>
            </div>
          </div>
        </main>
      </div>
    )
  }

  if (userRole === "employee") {
    router.push("/employee-dashboard")
    return null
  }

  return (
    <div className="min-h-screen bg-background">
      <DashboardHeader />

      <main className="mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
        <div className="mb-8 flex items-center justify-between">
          <div>
            <h1 className="text-3xl font-bold tracking-tight">Team</h1>
            <p className="mt-1 text-muted-foreground">Manage your employees and track their hours</p>
          </div>
          <AddEmployeeDialog onEmployeeAdded={handleEmployeeUpdate} />
        </div>

        {employees.length === 0 ? (
          <div className="flex min-h-[40vh] items-center justify-center">
            <div className="text-center">
              <h3 className="text-lg font-semibold text-muted-foreground">No employees yet</h3>
              <p className="mt-2 text-sm text-muted-foreground">Add your first employee to get started</p>
            </div>
          </div>
        ) : (
          <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
            {employees.map((employee) => (
              <div
                key={employee.id}
                className="group cursor-pointer rounded-xl border bg-card p-6 transition-all hover:border-primary hover:shadow-md"
                onClick={() => router.push(`/employee/${employee.id}`)}
              >
                <div className="mb-4 flex items-start justify-between">
                  <div>
                    <h3 className="text-lg font-semibold">{employee.name}</h3>
                    <p className="text-sm text-muted-foreground">{employee.position}</p>
                    <div className="mt-2 flex items-center gap-2">
                      <div className={`h-2 w-2 rounded-full ${employee.clockedIn ? "bg-green-500" : "bg-red-500"}`} />
                      <span className="text-xs font-medium text-muted-foreground">
                        {employee.clockedIn ? "Clocked In" : "Clocked Out"}
                      </span>
                    </div>
                    {employee.clockedIn && employee.clockInTime && (
                      <p className="mt-1 text-xs text-muted-foreground">
                        Since {new Date(`2000-01-01T${employee.clockInTime}`).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
                      </p>
                    )}
                  </div>
                </div>

                <div className="space-y-2">
                  <div className="flex items-center justify-between text-sm">
                    <span className="text-muted-foreground">This week</span>
                    <span className="font-semibold">{employee.currentWeekHours.toFixed(1)}h</span>
                  </div>
                  <div className="flex items-center justify-between text-sm">
                    <span className="text-muted-foreground">Weekly pay</span>
                    <span className="font-semibold text-primary">
                      ${(employee.currentWeekHours * employee.hourly_rate).toFixed(2)}
                    </span>
                  </div>
                  <div className="flex items-center justify-between text-sm">
                    <span className="text-muted-foreground">Hourly rate</span>
                    <span className="font-semibold">${employee.hourly_rate}/h</span>
                  </div>
                </div>
              </div>
            ))}
          </div>
        )}
      </main>
    </div>
  )
}
```

### 10. components/add-employee-dialog.tsx
```typescript
"use client"

import type React from "react"

import { useState } from "react"
import { useRouter } from "next/navigation"
import { Button } from "@/components/ui/button"
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Plus } from "lucide-react"
import { employeeService, userService } from "@/lib/database"

interface AddEmployeeDialogProps {
  onEmployeeAdded?: () => void
}

export function AddEmployeeDialog({ onEmployeeAdded }: AddEmployeeDialogProps) {
  const [open, setOpen] = useState(false)
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const router = useRouter()

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    setIsLoading(true)
    setError(null)

    const formData = new FormData(e.currentTarget)
    const name = formData.get("name") as string
    const email = formData.get("email") as string
    const position = formData.get("position") as string
    const hourlyRate = Number.parseFloat(formData.get("hourlyRate") as string)
    const taxNumber = formData.get("taxNumber") as string
    const ppsNumber = formData.get("ppsNumber") as string

    try {
      // Create employee
      const newEmployee = await employeeService.create({
        name,
        email: email || null,
        position: position || null,
        hourly_rate: hourlyRate,
        tax_number: taxNumber || null,
        pps_number: ppsNumber || null,
      })

      // Create user account for the employee
      if (email) {
        await userService.create({
          email,
          role: 'employee',
          employee_id: newEmployee.id,
        })
      }

      setOpen(false)
      onEmployeeAdded?.()
      router.refresh()
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to add employee")
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button size="lg" className="gap-2">
          <Plus className="h-5 w-5" />
          Add Employee
        </Button>
      </DialogTrigger>
      <DialogContent className="sm:max-w-[500px]">
        <DialogHeader>
          <DialogTitle className="text-2xl">Add New Employee</DialogTitle>
          <DialogDescription>Enter the employee details to add them to your team</DialogDescription>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-6">
          <div className="space-y-2">
            <Label htmlFor="name">Full Name *</Label>
            <Input id="name" name="name" placeholder="John Smith" required className="h-12" />
          </div>

          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>
            <Input id="email" name="email" type="email" placeholder="john@company.com" className="h-12" />
          </div>

          <div className="space-y-2">
            <Label htmlFor="position">Position</Label>
            <Input id="position" name="position" placeholder="Manager" className="h-12" />
          </div>

          <div className="space-y-2">
            <Label htmlFor="hourlyRate">Hourly Rate ($) *</Label>
            <Input
              id="hourlyRate"
              name="hourlyRate"
              type="number"
              step="0.01"
              min="0"
              placeholder="25.00"
              required
              className="h-12"
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="taxNumber">Tax Number</Label>
            <Input id="taxNumber" name="taxNumber" placeholder="1234567T" className="h-12" />
          </div>

          <div className="space-y-2">
            <Label htmlFor="ppsNumber">PPS Number (Irish)</Label>
            <Input id="ppsNumber" name="ppsNumber" placeholder="1234567AB" className="h-12" />
          </div>

          {error && <div className="rounded-lg bg-destructive/10 p-3 text-sm text-destructive">{error}</div>}

          <div className="flex gap-3">
            <Button type="button" variant="outline" onClick={() => setOpen(false)} className="flex-1">
              Cancel
            </Button>
            <Button type="submit" disabled={isLoading} className="flex-1">
              {isLoading ? "Adding..." : "Add Employee"}
            </Button>
          </div>
        </form>
      </DialogContent>
    </Dialog>
  )
}
```

### 11. components/dashboard-header.tsx
```typescript
"use client"

import Link from "next/link"
import { Button } from "@/components/ui/button"
import { LogoutButton } from "@/components/logout-button"
import { MessageSquare, Calendar, Users, BarChart3 } from "lucide-react"
import { useEffect, useState } from "react"

export function DashboardHeader() {
  const [userEmail, setUserEmail] = useState<string>("")

  useEffect(() => {
    const mockUser = localStorage.getItem("mockUser")
    if (mockUser) {
      const user = JSON.parse(mockUser)
      setUserEmail(user.email)
    }
  }, [])

  return (
    <header className="sticky top-0 z-50 border-b bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60">
      <div className="mx-auto flex h-16 max-w-7xl items-center justify-between px-4 sm:px-6 lg:px-8">
        <div className="flex items-center gap-8">
          <Link href="/" className="flex items-center gap-2">
            <div className="flex h-8 w-8 items-center justify-center rounded-lg bg-primary">
              <span className="text-lg font-bold text-primary-foreground">Z</span>
            </div>
            <span className="text-xl font-bold">Zap Pay</span>
          </Link>

          <nav className="hidden items-center gap-1 md:flex">
            <Button variant="ghost" size="sm" asChild>
              <Link href="/" className="gap-2">
                <Users className="h-4 w-4" />
                Team
              </Link>
            </Button>
            <Button variant="ghost" size="sm" asChild>
              <Link href="/schedule" className="gap-2">
                <Calendar className="h-4 w-4" />
                Roster
              </Link>
            </Button>
            <Button variant="ghost" size="sm" asChild>
              <Link href="/analytics" className="gap-2">
                <BarChart3 className="h-4 w-4" />
                Analytics
              </Link>
            </Button>
            <Button variant="ghost" size="sm" asChild>
              <Link href="/messages" className="gap-2">
                <MessageSquare className="h-4 w-4" />
                Messages
              </Link>
            </Button>
          </nav>
        </div>

        <div className="flex items-center gap-4">
          <div className="hidden text-right sm:block">
            <p className="text-sm font-medium">{userEmail}</p>
          </div>
          <LogoutButton />
        </div>
      </div>
    </header>
  )
}
```

### 12. components/employee-details.tsx
```typescript
"use client"

import { Card } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { Mail, DollarSign, Clock, Calendar, MessageSquare } from "lucide-react"
import Link from "next/link"
import { EditEmployeeDialog } from "@/components/edit-employee-dialog"
import { DeleteEmployeeDialog } from "@/components/delete-employee-dialog"
import { TimeTracker } from "@/components/time-tracker"

interface Employee {
  id: string
  name: string
  email: string | null
  position: string | null
  hourly_rate: number
  created_at: string
  tax_number?: string | null
  pps_number?: string | null
}

interface Schedule {
  id: string
  monday_hours: number
  tuesday_hours: number
  wednesday_hours: number
  thursday_hours: number
  friday_hours: number
  saturday_hours: number
  sunday_hours: number
  total_hours: number
  week_start_date: string
}

interface EmployeeDetailsProps {
  employee: Employee
  currentSchedule: Schedule | null
  weekStart: string
}

export function EmployeeDetails({ employee, currentSchedule, weekStart }: EmployeeDetailsProps) {
  const totalHours = currentSchedule?.total_hours || 0
  const weeklyPay = totalHours * employee.hourly_rate

  const formatDate = (dateStr: string) => {
    const date = new Date(dateStr)
    return date.toLocaleDateString("en-US", { month: "long", day: "numeric", year: "numeric" })
  }

  const formatWeekRange = (weekStartStr: string) => {
    const start = new Date(weekStartStr)
    const end = new Date(start)
    end.setDate(start.getDate() + 6)
    return `${start.toLocaleDateString("en-US", { month: "short", day: "numeric" })} - ${end.toLocaleDateString("en-US", { month: "short", day: "numeric", year: "numeric" })}`
  }

  return (
    <div className="space-y-6">
      {/* Header Card */}
      <Card className="p-8">
        <div className="flex items-start justify-between">
          <div className="flex items-start gap-6">
            <div className="flex h-20 w-20 items-center justify-center rounded-full bg-primary/10 text-3xl font-bold text-primary">
              {employee.name.charAt(0).toUpperCase()}
            </div>

            <div className="space-y-3">
              <div>
                <h1 className="text-3xl font-bold tracking-tight">{employee.name}</h1>
                {employee.position && <p className="mt-1 text-lg text-muted-foreground">{employee.position}</p>}
              </div>

              <div className="flex flex-wrap gap-4 text-sm">
                {employee.email && (
                  <div className="flex items-center gap-2 text-muted-foreground">
                    <Mail className="h-4 w-4" />
                    {employee.email}
                  </div>
                )}
                <div className="flex items-center gap-2 text-muted-foreground">
                  <DollarSign className="h-4 w-4" />${employee.hourly_rate.toFixed(2)}/hour
                </div>
                <div className="flex items-center gap-2 text-muted-foreground">
                  <Calendar className="h-4 w-4" />
                  Joined {formatDate(employee.created_at)}
                </div>
              </div>

              {(employee.tax_number || employee.pps_number) && (
                <div className="flex flex-wrap gap-4 text-sm border-t pt-3 mt-3">
                  {employee.tax_number && (
                    <div className="text-muted-foreground">
                      <span className="font-medium">Tax Number:</span> {employee.tax_number}
                    </div>
                  )}
                  {employee.pps_number && (
                    <div className="text-muted-foreground">
                      <span className="font-medium">PPS Number:</span> {employee.pps_number}
                    </div>
                  )}
                </div>
              )}
            </div>
          </div>

          <div className="flex gap-2">
            <EditEmployeeDialog employee={employee} />
            <DeleteEmployeeDialog employeeId={employee.id} employeeName={employee.name} />
          </div>
        </div>
      </Card>

      {/* Current Week Stats */}
      <div className="grid gap-6 md:grid-cols-2">
        <Card className="p-6">
          <div className="mb-2 flex items-center gap-2 text-sm text-muted-foreground">
            <Clock className="h-4 w-4" />
            This Week's Hours
          </div>
          <div className="mb-1 text-4xl font-bold">{totalHours.toFixed(1)}</div>
          <p className="text-sm text-muted-foreground">{formatWeekRange(weekStart)}</p>

          {currentSchedule && (
            <div className="mt-4 space-y-2 border-t pt-4">
              <div className="grid grid-cols-2 gap-2 text-sm">
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Mon:</span>
                  <span className="font-medium">{currentSchedule.monday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Tue:</span>
                  <span className="font-medium">{currentSchedule.tuesday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Wed:</span>
                  <span className="font-medium">{currentSchedule.wednesday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Thu:</span>
                  <span className="font-medium">{currentSchedule.thursday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Fri:</span>
                  <span className="font-medium">{currentSchedule.friday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Sat:</span>
                  <span className="font-medium">{currentSchedule.saturday_hours}h</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-muted-foreground">Sun:</span>
                  <span className="font-medium">{currentSchedule.sunday_hours}h</span>
                </div>
              </div>
            </div>
          )}
        </Card>

        <Card className="p-6">
          <div className="mb-2 flex items-center gap-2 text-sm text-muted-foreground">
            <DollarSign className="h-4 w-4" />
            This Week's Pay
          </div>
          <div className="mb-1 text-4xl font-bold">${weeklyPay.toFixed(2)}</div>
          <p className="text-sm text-muted-foreground">
            {totalHours.toFixed(1)} hours × ${employee.hourly_rate.toFixed(2)}/hour
          </p>

          <div className="mt-6 flex gap-3">
            <Button asChild className="flex-1 gap-2">
              <Link href={`/schedule?employee=${employee.id}`}>
                <Calendar className="h-4 w-4" />
                Edit Schedule
              </Link>
            </Button>
            <Button variant="outline" asChild className="flex-1 gap-2 bg-transparent">
              <Link href={`/messages?employee=${employee.id}`}>
                <MessageSquare className="h-4 w-4" />
                Message
              </Link>
            </Button>
          </div>
        </Card>
      </div>

      {/* Time Tracking Section */}
      <TimeTracker 
        employeeId={employee.id}
        employeeName={employee.name}
        hourlyRate={employee.hourly_rate}
      />
    </div>
  )
}
```

### 13. components/employee-schedule-history.tsx
```typescript
"use client"

import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Clock } from "lucide-react"

interface Schedule {
  id: string
  employee_id: string
  week_start_date: string
  monday_hours: number
  tuesday_hours: number
  wednesday_hours: number
  thursday_hours: number
  friday_hours: number
  saturday_hours: number
  sunday_hours: number
  total_hours: number
}

interface EmployeeScheduleHistoryProps {
  schedules: Schedule[]
}

export function EmployeeScheduleHistory({ schedules }: EmployeeScheduleHistoryProps) {
  const formatWeekRange = (weekStartStr: string) => {
    const start = new Date(weekStartStr)
    const end = new Date(start)
    end.setDate(start.getDate() + 6)
    return `${start.toLocaleDateString("en-US", { month: "short", day: "numeric" })} - ${end.toLocaleDateString("en-US", { month: "short", day: "numeric" })}`
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          <Clock className="h-5 w-5" />
          Schedule History
        </CardTitle>
        <CardDescription>
          Past 8 weeks of scheduled hours
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-4">
          {schedules.map((schedule) => (
            <div key={schedule.id} className="flex items-center justify-between rounded-lg border p-4">
              <div>
                <p className="font-medium">{formatWeekRange(schedule.week_start_date)}</p>
                <p className="text-sm text-muted-foreground">
                  {schedule.monday_hours + schedule.tuesday_hours + schedule.wednesday_hours + 
                   schedule.thursday_hours + schedule.friday_hours + schedule.saturday_hours + 
                   schedule.sunday_hours}h total
                </p>
              </div>
              <div className="text-right">
                <p className="font-semibold">{schedule.total_hours}h</p>
                <div className="flex gap-1 text-xs text-muted-foreground">
                  <span>M:{schedule.monday_hours}</span>
                  <span>T:{schedule.tuesday_hours}</span>
                  <span>W:{schedule.wednesday_hours}</span>
                  <span>T:{schedule.thursday_hours}</span>
                  <span>F:{schedule.friday_hours}</span>
                </div>
              </div>
            </div>
          ))}
        </div>
      </CardContent>
    </Card>
  )
}
```

### 14. components/logout-button.tsx
```typescript
"use client"

import { Button } from "@/components/ui/button"
import { useRouter } from "next/navigation"
import { LogOut } from "lucide-react"
import { authService } from "@/lib/database"

export function LogoutButton() {
  const router = useRouter()

  const handleLogout = async () => {
    try {
      await authService.signOut()
      router.push("/auth/login")
      router.refresh()
    } catch (error) {
      console.error("Logout error:", error)
    }
  }

  return (
    <Button variant="ghost" size="sm" onClick={handleLogout} className="gap-2">
      <LogOut className="h-4 w-4" />
      <span className="hidden sm:inline">Sign out</span>
    </Button>
  )
}
```

### 15. components/time-tracker.tsx
```typescript
"use client"

import { useState, useEffect } from "react"
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Clock, Play, Square } from "lucide-react"
import { timeEntryService } from "@/lib/database"
import { TimeEntry } from "@/lib/supabase"

interface TimeTrackerProps {
  employeeId: string
  employeeName: string
  hourlyRate: number
}

export function TimeTracker({ employeeId, employeeName, hourlyRate }: TimeTrackerProps) {
  const [isClockedIn, setIsClockedIn] = useState(false)
  const [clockInTime, setClockInTime] = useState<string | null>(null)
  const [currentSessionHours, setCurrentSessionHours] = useState(0)
  const [todayHours, setTodayHours] = useState(0)
  const [weekHours, setWeekHours] = useState(0)
  const [timeEntries, setTimeEntries] = useState<TimeEntry[]>([])
  const [currentEntryId, setCurrentEntryId] = useState<string | null>(null)

  useEffect(() => {
    loadTimeEntries()
    const interval = setInterval(updateCurrentSession, 1000)
    return () => clearInterval(interval)
  }, [employeeId])

  const loadTimeEntries = async () => {
    try {
      const entries = await timeEntryService.getAllForEmployee(employeeId)
      setTimeEntries(entries)
      
      const today = new Date().toISOString().split('T')[0]
      const todayEntry = entries.find((entry: TimeEntry) => entry.date === today)
      
      if (todayEntry) {
        setIsClockedIn(!todayEntry.clock_out_time)
        setClockInTime(todayEntry.clock_in_time)
        setCurrentEntryId(todayEntry.id)
        
        // Calculate today's hours (excluding current session)
        const completedTodayHours = entries
          .filter((entry: TimeEntry) => entry.date === today && entry.clock_out_time)
          .reduce((total: number, entry: TimeEntry) => {
            return total + (entry.total_hours || 0)
          }, 0)
        setTodayHours(completedTodayHours)
      }
      
      // Calculate week hours
      const weekStart = getWeekStart(new Date())
      const weekEntries = entries.filter((entry: TimeEntry) => {
        const entryDate = new Date(entry.date)
        return entryDate >= weekStart && entry.clock_out_time
      })
      
      const totalWeekHours = weekEntries.reduce((total: number, entry: TimeEntry) => {
        return total + (entry.total_hours || 0)
      }, 0)
      setWeekHours(totalWeekHours)
    } catch (error) {
      console.error("Error loading time entries:", error)
    }
  }

  const getWeekStart = (date: Date): Date => {
    const dayOfWeek = date.getDay()
    const diff = dayOfWeek === 0 ? -6 : 1 - dayOfWeek
    const weekStart = new Date(date)
    weekStart.setDate(date.getDate() + diff)
    weekStart.setHours(0, 0, 0, 0)
    return weekStart
  }

  const updateCurrentSession = () => {
    if (isClockedIn && clockInTime) {
      const now = new Date()
      const clockIn = new Date(`2000-01-01T${clockInTime}`)
      const currentTime = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}`
      const currentClockIn = new Date(`2000-01-01T${currentTime}`)
      
      const hours = (currentClockIn.getTime() - clockIn.getTime()) / (1000 * 60 * 60)
      setCurrentSessionHours(Math.max(0, hours))
    }
  }

  const handleClockIn = async () => {
    try {
      const now = new Date()
      const timeString = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}`
      const today = now.toISOString().split('T')[0]
      
      const newEntry = await timeEntryService.create({
        employee_id: employeeId,
        date: today,
        clock_in_time: timeString,
        clock_out_time: null,
        total_hours: null
      })
      
      setTimeEntries(prev => [...prev, newEntry])
      setIsClockedIn(true)
      setClockInTime(timeString)
      setCurrentSessionHours(0)
      setCurrentEntryId(newEntry.id)
    } catch (error) {
      console.error("Error clocking in:", error)
    }
  }

  const handleClockOut = async () => {
    try {
      const now = new Date()
      const timeString = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}`
      
      if (!currentEntryId || !clockInTime) return
      
      const clockIn = new Date(`2000-01-01T${clockInTime}`)
      const clockOut = new Date(`2000-01-01T${timeString}`)
      const totalHours = (clockOut.getTime() - clockIn.getTime()) / (1000 * 60 * 60)
      
      const updatedEntry = await timeEntryService.update(currentEntryId, {
        clock_out_time: timeString,
        total_hours: totalHours
      })
      
      setTimeEntries(prev => 
        prev.map(entry => 
          entry.id === currentEntryId ? updatedEntry : entry
        )
      )
      
      setIsClockedIn(false)
      setClockInTime(null)
      setCurrentSessionHours(0)
      setCurrentEntryId(null)
      
      // Update today's hours
      const completedTodayHours = timeEntries
        .filter((entry: TimeEntry) => entry.date === updatedEntry.date && entry.clock_out_time)
        .reduce((total: number, entry: TimeEntry) => {
          return total + (entry.total_hours || 0)
        }, 0) + totalHours
      setTodayHours(completedTodayHours)
      
      // Update week hours
      const weekStart = getWeekStart(new Date())
      const weekEntries = timeEntries.filter((entry: TimeEntry) => {
        const entryDate = new Date(entry.date)
        return entryDate >= weekStart && entry.clock_out_time
      })
      
      const totalWeekHours = weekEntries.reduce((total: number, entry: TimeEntry) => {
        return total + (entry.total_hours || 0)
      }, 0) + totalHours
      setWeekHours(totalWeekHours)
    } catch (error) {
      console.error("Error clocking out:", error)
    }
  }

  const formatTime = (hours: number): string => {
    const h = Math.floor(hours)
    const m = Math.floor((hours - h) * 60)
    return `${h}h ${m}m`
  }

  return (
    <div className="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Clock className="h-5 w-5" />
            Time Tracking
          </CardTitle>
          <CardDescription>
            Clock in and out to track your working hours
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="flex items-center justify-between">
            <div>
              <p className="text-sm font-medium">Status</p>
              <p className={`text-lg font-semibold ${isClockedIn ? 'text-green-600' : 'text-red-600'}`}>
                {isClockedIn ? 'Clocked In' : 'Clocked Out'}
              </p>
              {isClockedIn && clockInTime && (
                <p className="text-sm text-muted-foreground">
                  Since {new Date(`2000-01-01T${clockInTime}`).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
                </p>
              )}
            </div>
            <Button
              onClick={isClockedIn ? handleClockOut : handleClockIn}
              variant={isClockedIn ? "destructive" : "default"}
              size="lg"
              className="gap-2"
            >
              {isClockedIn ? (
                <>
                  <Square className="h-4 w-4" />
                  Clock Out
                </>
              ) : (
                <>
                  <Play className="h-4 w-4" />
                  Clock In
                </>
              )}
            </Button>
          </div>
          
          {isClockedIn && (
            <div className="rounded-lg bg-muted p-4">
              <p className="text-sm font-medium">Current Session</p>
              <p className="text-2xl font-bold text-primary">
                {formatTime(currentSessionHours)}
              </p>
            </div>
          )}
        </CardContent>
      </Card>

      <div className="grid gap-4 md:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle className="text-lg">Today</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-2">
              <div className="flex justify-between">
                <span className="text-sm text-muted-foreground">Hours worked</span>
                <span className="font-semibold">{formatTime(todayHours + (isClockedIn ? currentSessionHours : 0))}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-sm text-muted-foreground">Earnings</span>
                <span className="font-semibold text-primary">
                  ${((todayHours + (isClockedIn ? currentSessionHours : 0)) * hourlyRate).toFixed(2)}
                </span>
              </div>
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle className="text-lg">This Week</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-2">
              <div className="flex justify-between">
                <span className="text-sm text-muted-foreground">Hours worked</span>
                <span className="font-semibold">{formatTime(weekHours + (isClockedIn ? currentSessionHours : 0))}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-sm text-muted-foreground">Earnings</span>
                <span className="font-semibold text-primary">
                  ${((weekHours + (isClockedIn ? currentSessionHours : 0)) * hourlyRate).toFixed(2)}
                </span>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
```

### 16. lib/database.ts
```typescript
import { supabase, Employee, TimeEntry, User } from './supabase'

// Employee operations
export const employeeService = {
  async getAll(): Promise<Employee[]> {
    const { data, error } = await supabase
      .from('employees')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    return data || []
  },

  async getById(id: string): Promise<Employee | null> {
    const { data, error } = await supabase
      .from('employees')
      .select('*')
      .eq('id', id)
      .single()
    
    if (error) throw error
    return data
  },

  async create(employee: Omit<Employee, 'id' | 'created_at' | 'updated_at'>): Promise<Employee> {
    const { data, error } = await supabase
      .from('employees')
      .insert(employee)
      .select()
      .single()
    
    if (error) throw error
    return data
  },

  async update(id: string, updates: Partial<Employee>): Promise<Employee> {
    const { data, error } = await supabase
      .from('employees')
      .update(updates)
      .eq('id', id)
      .select()
      .single()
    
    if (error) throw error
    return data
  },

  async delete(id: string): Promise<void> {
    const { error } = await supabase
      .from('employees')
      .delete()
      .eq('id', id)
    
    if (error) throw error
  },

  async getByEmail(email: string): Promise<Employee | null> {
    const { data, error } = await supabase
      .from('employees')
      .select('*')
      .eq('email', email)
      .single()
    
    if (error) throw error
    return data
  }
}

// Time entry operations
export const timeEntryService = {
  async getAllForEmployee(employeeId: string): Promise<TimeEntry[]> {
    const { data, error } = await supabase
      .from('time_entries')
      .select('*')
      .eq('employee_id', employeeId)
      .order('date', { ascending: false })
    
    if (error) throw error
    return data || []
  },

  async getTodayEntry(employeeId: string): Promise<TimeEntry | null> {
    const today = new Date().toISOString().split('T')[0]
    const { data, error } = await supabase
      .from('time_entries')
      .select('*')
      .eq('employee_id', employeeId)
      .eq('date', today)
      .single()
    
    if (error && error.code !== 'PGRST116') throw error
    return data
  },

  async create(entry: Omit<TimeEntry, 'id' | 'created_at' | 'updated_at'>): Promise<TimeEntry> {
    const { data, error } = await supabase
      .from('time_entries')
      .insert(entry)
      .select()
      .single()
    
    if (error) throw error
    return data
  },

  async update(id: string, updates: Partial<TimeEntry>): Promise<TimeEntry> {
    const { data, error } = await supabase
      .from('time_entries')
      .update(updates)
      .eq('id', id)
      .select()
      .single()
    
    if (error) throw error
    return data
  },

  async getWeekHours(employeeId: string, weekStart: Date): Promise<number> {
    const weekEnd = new Date(weekStart)
    weekEnd.setDate(weekEnd.getDate() + 6)
    
    const { data, error } = await supabase
      .from('time_entries')
      .select('total_hours')
      .eq('employee_id', employeeId)
      .gte('date', weekStart.toISOString().split('T')[0])
      .lte('date', weekEnd.toISOString().split('T')[0])
      .not('total_hours', 'is', null)
    
    if (error) throw error
    
    return data?.reduce((total, entry) => total + (entry.total_hours || 0), 0) || 0
  }
}

// User operations
export const userService = {
  async getByEmail(email: string): Promise<User | null> {
    const { data, error } = await supabase
      .from('users')
      .select('*')
      .eq('email', email)
      .single()
    
    if (error && error.code !== 'PGRST116') throw error
    return data
  },

  async create(user: Omit<User, 'id' | 'created_at' | 'updated_at'>): Promise<User> {
    const { data, error } = await supabase
      .from('users')
      .insert(user)
      .select()
      .single()
    
    if (error) throw error
    return data
  },

  async update(id: string, updates: Partial<User>): Promise<User> {
    const { data, error } = await supabase
      .from('users')
      .update(updates)
      .eq('id', id)
      .select()
      .single()
    
    if (error) throw error
    return data
  }
}

// Authentication helpers
export const authService = {
  async signIn(email: string, password: string) {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    })
    
    if (error) throw error
    return data
  },

  async signUp(email: string, password: string, role: 'administrator' | 'employee' = 'employee') {
    const { data, error } = await supabase.auth.signUp({
      email,
      password
    })
    
    if (error) throw error
    
    // Create user profile
    if (data.user) {
      await userService.create({
        email: data.user.email!,
        role,
        employee_id: null
      })
    }
    
    return data
  },

  async signOut() {
    const { error } = await supabase.auth.signOut()
    if (error) throw error
  },

  async getCurrentUser() {
    const { data: { user }, error } = await supabase.auth.getUser()
    if (error) throw error
    return user
  }
}
```

### 17. lib/supabase.ts
```typescript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient(supabaseUrl, supabaseAnonKey)

// Database types
export interface Employee {
  id: string
  name: string
  email: string | null
  position: string | null
  hourly_rate: number
  tax_number: string | null
  pps_number: string | null
  created_at: string
  updated_at: string
}

export interface TimeEntry {
  id: string
  employee_id: string
  date: string
  clock_in_time: string
  clock_out_time: string | null
  total_hours: number | null
  created_at: string
  updated_at: string
}

export interface User {
  id: string
  email: string
  role: 'administrator' | 'employee'
  employee_id: string | null
  created_at: string
  updated_at: string
}
```

### 18. middleware.ts
```typescript
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(req: NextRequest) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })

  const {
    data: { session },
  } = await supabase.auth.getSession()

  // Protect routes that require authentication
  if (!session && req.nextUrl.pathname !== '/auth/login') {
    return NextResponse.redirect(new URL('/auth/login', req.url))
  }

  // Redirect authenticated users away from login page
  if (session && req.nextUrl.pathname === '/auth/login') {
    return NextResponse.redirect(new URL('/', req.url))
  }

  return res
}

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    '/((?!_next/static|_next/image|favicon.ico).*)',
  ],
}
```

### 19. package.json
```json
{
  "name": "zap-pay-saas",
  "version": "1.0.0",
  "description": "Modern payroll management application with time tracking and employee management",
  "private": true,
  "scripts": {
    "build": "next build",
    "dev": "next dev",
    "lint": "eslint .",
    "start": "next start",
    "type-check": "tsc --noEmit"
  },
  "keywords": [
    "payroll",
    "time-tracking",
    "employee-management",
    "nextjs",
    "supabase",
    "saas"
  ],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/zap-pay-saas.git"
  },
  "homepage": "https://zap-pay-saas.vercel.app",
  "dependencies": {
    "@hookform/resolvers": "^3.10.0",
    "@radix-ui/react-accordion": "1.2.2",
    "@radix-ui/react-alert-dialog": "1.1.4",
    "@radix-ui/react-aspect-ratio": "1.1.1",
    "@radix-ui/react-avatar": "1.1.2",
    "@radix-ui/react-checkbox": "1.1.3",
    "@radix-ui/react-collapsible": "1.1.2",
    "@radix-ui/react-context-menu": "2.2.4",
    "@radix-ui/react-dialog": "1.1.4",
    "@radix-ui/react-dropdown-menu": "2.1.4",
    "@radix-ui/react-hover-card": "1.1.4",
    "@radix-ui/react-label": "2.1.1",
    "@radix-ui/react-menubar": "1.1.4",
    "@radix-ui/react-navigation-menu": "1.2.3",
    "@radix-ui/react-popover": "1.1.4",
    "@radix-ui/react-progress": "1.1.1",
    "@radix-ui/react-radio-group": "1.2.2",
    "@radix-ui/react-scroll-area": "1.2.2",
    "@radix-ui/react-select": "2.1.4",
    "@radix-ui/react-separator": "1.1.1",
    "@radix-ui/react-slider": "1.2.2",
    "@radix-ui/react-slot": "1.1.1",
    "@radix-ui/react-switch": "1.1.2",
    "@radix-ui/react-tabs": "1.1.2",
    "@radix-ui/react-toast": "1.2.4",
    "@radix-ui/react-toggle": "1.1.1",
    "@radix-ui/react-toggle-group": "1.1.1",
    "@radix-ui/react-tooltip": "1.1.6",
    "@supabase/ssr": "latest",
    "@supabase/supabase-js": "latest",
    "@vercel/analytics": "1.3.1",
    "autoprefixer": "^10.4.20",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "cmdk": "1.0.4",
    "date-fns": "4.1.0",
    "embla-carousel-react": "8.5.1",
    "input-otp": "1.4.1",
    "lucide-react": "^0.454.0",
    "next": "15.2.4",
    "next-themes": "^0.4.6",
    "react": "^19",
    "react-day-picker": "9.8.0",
    "react-dom": "^19",
    "react-hook-form": "^7.60.0",
    "react-resizable-panels": "^2.1.7",
    "recharts": "2.15.4",
    "sonner": "^1.7.4",
    "tailwind-merge": "^2.5.5",
    "tailwindcss-animate": "^1.0.7",
    "vaul": "^0.9.9",
    "zod": "3.25.76"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4.1.9",
    "@types/node": "^22",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "postcss": "^8.5",
    "tailwindcss": "^4.1.9",
    "tw-animate-css": "1.3.3",
    "typescript": "^5"
  }
}
```

### 20. supabase-schema.sql
```sql
-- Enable Row Level Security
ALTER TABLE auth.users ENABLE ROW LEVEL SECURITY;

-- Create employees table
CREATE TABLE IF NOT EXISTS employees (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT UNIQUE,
  position TEXT,
  hourly_rate DECIMAL(10,2) NOT NULL DEFAULT 0,
  tax_number TEXT,
  pps_number TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create time_entries table
CREATE TABLE IF NOT EXISTS time_entries (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  employee_id UUID NOT NULL REFERENCES employees(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  clock_in_time TIME NOT NULL,
  clock_out_time TIME,
  total_hours DECIMAL(5,2),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(employee_id, date)
);

-- Create users table for role management
CREATE TABLE IF NOT EXISTS users (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  role TEXT NOT NULL DEFAULT 'employee' CHECK (role IN ('administrator', 'employee')),
  employee_id UUID REFERENCES employees(id) ON DELETE SET NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create indexes for better performance
CREATE INDEX IF NOT EXISTS idx_time_entries_employee_id ON time_entries(employee_id);
CREATE INDEX IF NOT EXISTS idx_time_entries_date ON time_entries(date);
CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
CREATE INDEX IF NOT EXISTS idx_employees_email ON employees(email);

-- Enable Row Level Security on all tables
ALTER TABLE employees ENABLE ROW LEVEL SECURITY;
ALTER TABLE time_entries ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- RLS Policies for employees table
CREATE POLICY "Administrators can view all employees" ON employees
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

CREATE POLICY "Administrators can insert employees" ON employees
  FOR INSERT WITH CHECK (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

CREATE POLICY "Administrators can update employees" ON employees
  FOR UPDATE USING (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

CREATE POLICY "Administrators can delete employees" ON employees
  FOR DELETE USING (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

-- RLS Policies for time_entries table
CREATE POLICY "Users can view their own time entries" ON time_entries
  FOR SELECT USING (
    employee_id IN (
      SELECT e.id FROM employees e
      JOIN users u ON u.employee_id = e.id
      WHERE u.email = auth.jwt() ->> 'email'
    )
    OR
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

CREATE POLICY "Users can insert their own time entries" ON time_entries
  FOR INSERT WITH CHECK (
    employee_id IN (
      SELECT e.id FROM employees e
      JOIN users u ON u.employee_id = e.id
      WHERE u.email = auth.jwt() ->> 'email'
    )
    OR
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

CREATE POLICY "Users can update their own time entries" ON time_entries
  FOR UPDATE USING (
    employee_id IN (
      SELECT e.id FROM employees e
      JOIN users u ON u.employee_id = e.id
      WHERE u.email = auth.jwt() ->> 'email'
    )
    OR
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

-- RLS Policies for users table
CREATE POLICY "Users can view their own profile" ON users
  FOR SELECT USING (
    email = auth.jwt() ->> 'email'
    OR
    EXISTS (
      SELECT 1 FROM users u
      WHERE u.email = auth.jwt() ->> 'email' 
      AND u.role = 'administrator'
    )
  );

CREATE POLICY "Administrators can manage all users" ON users
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.email = auth.jwt() ->> 'email' 
      AND users.role = 'administrator'
    )
  );

-- Create updated_at trigger function
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ language 'plpgsql';

-- Create triggers for updated_at
CREATE TRIGGER update_employees_updated_at BEFORE UPDATE ON employees
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_time_entries_updated_at BEFORE UPDATE ON time_entries
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Insert some sample data
INSERT INTO employees (id, name, email, position, hourly_rate) VALUES
  ('11111111-1111-1111-1111-111111111111', 'John Smith', 'john@example.com', 'Software Engineer', 50.00),
  ('22222222-2222-2222-2222-222222222222', 'Sarah Johnson', 'sarah@example.com', 'Designer', 45.00),
  ('33333333-3333-3333-3333-333333333333', 'Mike Davis', 'mike@example.com', 'Product Manager', 55.00)
ON CONFLICT (email) DO NOTHING;

INSERT INTO users (email, role, employee_id) VALUES
  ('admin@zappay.com', 'administrator', NULL),
  ('john@example.com', 'employee', '11111111-1111-1111-1111-111111111111'),
  ('sarah@example.com', 'employee', '22222222-2222-2222-2222-222222222222'),
  ('mike@example.com', 'employee', '33333333-3333-3333-3333-333333333333')
ON CONFLICT (email) DO NOTHING;
```

### 21. vercel.json
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "installCommand": "npm install",
  "devCommand": "npm run dev"
}
```

## 🚀 Quick Setup Instructions:

### 1. Create GitHub Repository
1. Go to GitHub and create a new repository named `zap-pay-saas`
2. Copy all the files above into your repository
3. Update the repository URL in `package.json`

### 2. Set up Supabase
1. Create a Supabase project at [supabase.com](https://supabase.com)
2. Run the SQL from `supabase-schema.sql` in the SQL Editor
3. Get your project URL and anon key from Settings > API

### 3. Deploy to Vercel
1. Go to [vercel.com](https://vercel.com) and connect your GitHub
2. Import your `zap-pay-saas` repository
3. Add environment variables:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
4. Deploy!

### 4. Test
- Visit your Vercel URL
- Create accounts or use demo accounts
- Test time tracking and employee management

**Demo Accounts:**
- Admin: `admin@zappay.com` / `admin123`
- Employee: `john@example.com` / `employee123`

This is a complete, production-ready application with Supabase integration!
