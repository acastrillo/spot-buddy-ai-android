# Web App Reference for Android Development

**Purpose**: This document maps TypeScript code from the web app to Kotlin equivalents for the Android app.

**Key Principle**: The Android app does NOT reuse TypeScript code directly. Instead, you'll **port the business logic** and **match the data models** to ensure compatibility with the shared AWS backend.

---

## What You NEED from Web App

### ✅ 1. Data Models (TypeScript → Kotlin)

The Android app must match these **exact data structures** for API compatibility.

#### User Model

**TypeScript** (`src/lib/dynamodb.ts`):
```typescript
export interface DynamoDBUser {
  id: string;
  email: string;
  firstName?: string | null;
  lastName?: string | null;
  emailVerified?: string | null;
  image?: string | null;
  createdAt: string; // ISO timestamp
  updatedAt: string; // ISO timestamp

  // Subscription & Billing
  subscriptionTier: "free" | "starter" | "pro" | "elite";
  subscriptionStatus: "active" | "inactive" | "trialing" | "canceled" | "past_due";
  subscriptionStartDate?: string | null;
  subscriptionEndDate?: string | null;
  trialEndsAt?: string | null;
  stripeCustomerId?: string | null;
  stripeSubscriptionId?: string | null;

  // Usage Tracking & Quotas
  ocrQuotaUsed: number;
  ocrQuotaLimit: number;
  ocrQuotaResetDate?: string | null;
  workoutsSaved: number;

  // AI Features (Phase 6)
  aiRequestsUsed?: number;
  aiRequestsLimit?: number;
  lastAiRequestReset?: string | null;

  // Training Profile
  trainingProfile?: TrainingProfile;
  experience?: 'beginner' | 'intermediate' | 'advanced';
}
```

**Kotlin** (Android equivalent):
```kotlin
// File: app/src/main/java/com/spotbuddy/app/data/remote/dto/UserDto.kt
data class UserDto(
    val id: String,
    val email: String,
    val firstName: String? = null,
    val lastName: String? = null,
    val emailVerified: String? = null,
    val image: String? = null,
    val createdAt: String, // ISO timestamp
    val updatedAt: String,

    // Subscription & Billing
    val subscriptionTier: SubscriptionTier,
    val subscriptionStatus: SubscriptionStatus,
    val subscriptionStartDate: String? = null,
    val subscriptionEndDate: String? = null,
    val trialEndsAt: String? = null,
    val stripeCustomerId: String? = null,
    val stripeSubscriptionId: String? = null,

    // Usage Tracking
    val ocrQuotaUsed: Int,
    val ocrQuotaLimit: Int,
    val ocrQuotaResetDate: String? = null,
    val workoutsSaved: Int,

    // AI Features
    val aiRequestsUsed: Int? = null,
    val aiRequestsLimit: Int? = null,
    val lastAiRequestReset: String? = null,

    // Training Profile
    val trainingProfile: TrainingProfileDto? = null,
    val experience: ExperienceLevel? = null
)

enum class SubscriptionTier {
    @SerializedName("free") FREE,
    @SerializedName("starter") STARTER,
    @SerializedName("pro") PRO,
    @SerializedName("elite") ELITE
}

enum class SubscriptionStatus {
    @SerializedName("active") ACTIVE,
    @SerializedName("inactive") INACTIVE,
    @SerializedName("trialing") TRIALING,
    @SerializedName("canceled") CANCELED,
    @SerializedName("past_due") PAST_DUE
}

enum class ExperienceLevel {
    @SerializedName("beginner") BEGINNER,
    @SerializedName("intermediate") INTERMEDIATE,
    @SerializedName("advanced") ADVANCED
}
```

#### Workout Model

**TypeScript** (`src/lib/dynamodb.ts`):
```typescript
export interface DynamoDBWorkout {
  userId: string; // Partition key
  workoutId: string; // Sort key
  title: string;
  description?: string | null;
  exercises: DynamoDBExercise[];
  content: string; // Original caption/text
  author?: { username: string } | null;
  createdAt: string; // ISO timestamp
  updatedAt: string;
  source: string; // URL or 'manual'
  type: string; // 'url' or 'manual'
  totalDuration: number; // Minutes
  difficulty: string; // 'easy', 'moderate', 'hard'
  tags: string[];
  llmData?: any | null;
  imageUrls?: string[];
  thumbnailUrl?: string | null;

  // Scheduling & Status
  scheduledDate?: string | null; // ISO date (YYYY-MM-DD)
  status?: 'scheduled' | 'completed' | 'skipped' | null;
  completedDate?: string | null;

  // AI Enhancement
  aiEnhanced?: boolean;
  aiNotes?: string | null;
  muscleGroups?: string[];
}

export interface DynamoDBExercise {
  id: string;
  name: string;
  sets: number;
  reps: string | number;
  weight?: string | null;
  restSeconds?: number | null;
  notes?: string | null;
  setDetails?: Array<{
    id?: string | null;
    reps?: string | number | null;
    weight?: string | number | null;
  }> | null;
}
```

**Kotlin**:
```kotlin
// File: app/src/main/java/com/spotbuddy/app/data/remote/dto/WorkoutDto.kt
data class WorkoutDto(
    val userId: String,
    val workoutId: String,
    val title: String,
    val description: String? = null,
    val exercises: List<ExerciseDto>,
    val content: String,
    val author: AuthorDto? = null,
    val createdAt: String,
    val updatedAt: String,
    val source: String,
    val type: String,
    val totalDuration: Int,
    val difficulty: String,
    val tags: List<String> = emptyList(),
    val llmData: Any? = null,
    val imageUrls: List<String>? = null,
    val thumbnailUrl: String? = null,

    // Scheduling
    val scheduledDate: String? = null,
    val status: WorkoutStatus? = null,
    val completedDate: String? = null,

    // AI
    val aiEnhanced: Boolean? = null,
    val aiNotes: String? = null,
    val muscleGroups: List<String>? = null
)

data class ExerciseDto(
    val id: String,
    val name: String,
    val sets: Int,
    val reps: Any, // Can be String or Int
    val weight: String? = null,
    val restSeconds: Int? = null,
    val notes: String? = null,
    val setDetails: List<SetDetailDto>? = null
)

data class SetDetailDto(
    val id: String? = null,
    val reps: Any? = null, // Can be String or Int
    val weight: Any? = null
)

data class AuthorDto(
    val username: String
)

enum class WorkoutStatus {
    @SerializedName("scheduled") SCHEDULED,
    @SerializedName("completed") COMPLETED,
    @SerializedName("skipped") SKIPPED
}
```

#### Body Metrics Model

**TypeScript** (`src/lib/dynamodb-body-metrics.ts`):
```typescript
export interface DynamoDBBodyMetric {
  userId: string; // Partition key
  date: string; // Sort key (ISO date: YYYY-MM-DD)
  weight?: number | null;
  bodyFatPercentage?: number | null;
  muscleMass?: number | null;
  chest?: number | null;
  waist?: number | null;
  hips?: number | null;
  thighs?: number | null;
  arms?: number | null;
  calves?: number | null;
  shoulders?: number | null;
  neck?: number | null;
  unit: 'metric' | 'imperial';
  notes?: string | null;
  photoUrls?: string[];
  createdAt: string;
  updatedAt: string;
}
```

**Kotlin**:
```kotlin
data class BodyMetricDto(
    val userId: String,
    val date: String, // YYYY-MM-DD
    val weight: Double? = null,
    val bodyFatPercentage: Double? = null,
    val muscleMass: Double? = null,
    val chest: Double? = null,
    val waist: Double? = null,
    val hips: Double? = null,
    val thighs: Double? = null,
    val arms: Double? = null,
    val calves: Double? = null,
    val shoulders: Double? = null,
    val neck: Double? = null,
    val unit: MeasurementUnit,
    val notes: String? = null,
    val photoUrls: List<String>? = null,
    val createdAt: String,
    val updatedAt: String
)

enum class MeasurementUnit {
    @SerializedName("metric") METRIC,
    @SerializedName("imperial") IMPERIAL
}
```

---

### ✅ 2. Business Logic to Port (TypeScript → Kotlin)

These files contain **algorithms** you need to reimplement in Kotlin:

#### A. PR Calculator (`src/lib/pr-calculator.ts`)

**Purpose**: Calculate 1-rep max (1RM) using 7 different formulas

**Key Functions**:
- `calculate1RM()` - Calculates estimated 1RM from weight × reps
- 7 formulas: Brzycki, Epley, Lander, Lombardi, Mayhew, O'Conner, Wathan

**Kotlin Port**:
```kotlin
// File: app/src/main/java/com/spotbuddy/app/domain/usecase/Calculate1RMUseCase.kt
object PRCalculator {

    enum class Formula {
        BRZYCKI, EPLEY, LANDER, LOMBARDI,
        MAYHEW, OCONNER, WATHAN
    }

    fun calculate1RM(weight: Double, reps: Int, formula: Formula = Formula.BRZYCKI): Double {
        if (reps == 1) return weight

        return when (formula) {
            Formula.BRZYCKI -> weight * (36.0 / (37.0 - reps))
            Formula.EPLEY -> weight * (1 + reps / 30.0)
            Formula.LANDER -> (100 * weight) / (101.3 - 2.67123 * reps)
            Formula.LOMBARDI -> weight * Math.pow(reps.toDouble(), 0.1)
            Formula.MAYHEW -> (100 * weight) / (52.2 + 41.9 * Math.exp(-0.055 * reps))
            Formula.OCONNER -> weight * (1 + reps / 40.0)
            Formula.WATHAN -> (100 * weight) / (48.8 + 53.8 * Math.exp(-0.075 * reps))
        }
    }

    fun calculateAllFormulas(weight: Double, reps: Int): Map<Formula, Double> {
        return Formula.values().associateWith { calculate1RM(weight, reps, it) }
    }

    fun calculateAverage1RM(weight: Double, reps: Int): Double {
        val results = calculateAllFormulas(weight, reps)
        return results.values.average()
    }
}
```

**Reference**: See web app `src/lib/pr-calculator.ts` for complete implementation details

#### B. Exercise History Extraction (`src/lib/exercise-history.ts`)

**Purpose**: Extract exercise performance data from workouts for stats/charts

**Key Functions**:
- `extractExerciseHistory()` - Groups workouts by exercise name
- `categorizeMuscleGroup()` - Maps exercises to muscle groups

**Kotlin Port**:
```kotlin
// File: app/src/main/java/com/spotbuddy/app/domain/usecase/ExtractExerciseHistoryUseCase.kt
object ExerciseHistoryExtractor {

    data class ExerciseHistoryEntry(
        val exerciseName: String,
        val date: String,
        val sets: Int,
        val reps: Any,
        val weight: String?,
        val volume: Double, // weight × reps × sets
        val estimated1RM: Double?,
        val workoutId: String,
        val workoutTitle: String
    )

    fun extractHistory(workouts: List<Workout>): Map<String, List<ExerciseHistoryEntry>> {
        return workouts
            .flatMap { workout ->
                workout.exercises.map { exercise ->
                    ExerciseHistoryEntry(
                        exerciseName = exercise.name,
                        date = workout.createdAt,
                        sets = exercise.sets,
                        reps = exercise.reps,
                        weight = exercise.weight,
                        volume = calculateVolume(exercise),
                        estimated1RM = calculate1RMIfPossible(exercise),
                        workoutId = workout.workoutId,
                        workoutTitle = workout.title
                    )
                }
            }
            .groupBy { it.exerciseName }
            .mapValues { (_, entries) -> entries.sortedByDescending { it.date } }
    }

    fun categorizeMuscleGroup(exerciseName: String): String {
        val name = exerciseName.lowercase()
        return when {
            name.contains("bench") || name.contains("chest") || name.contains("push up") -> "Chest"
            name.contains("squat") || name.contains("lunge") || name.contains("leg") -> "Legs"
            name.contains("deadlift") || name.contains("row") || name.contains("pull") -> "Back"
            name.contains("shoulder") || name.contains("press") || name.contains("ohp") -> "Shoulders"
            name.contains("curl") || name.contains("tricep") || name.contains("bicep") -> "Arms"
            name.contains("plank") || name.contains("crunch") || name.contains("ab") -> "Core"
            else -> "Other"
        }
    }

    private fun calculateVolume(exercise: Exercise): Double {
        val weight = exercise.weight?.toDoubleOrNull() ?: 0.0
        val reps = when (val r = exercise.reps) {
            is Int -> r.toDouble()
            is String -> r.toIntOrNull()?.toDouble() ?: 0.0
            else -> 0.0
        }
        return weight * reps * exercise.sets
    }

    private fun calculate1RMIfPossible(exercise: Exercise): Double? {
        val weight = exercise.weight?.toDoubleOrNull() ?: return null
        val reps = when (val r = exercise.reps) {
            is Int -> r
            is String -> r.toIntOrNull()
            else -> null
        } ?: return null

        return PRCalculator.calculateAverage1RM(weight, reps)
    }
}
```

#### C. Smart Workout Parser (`src/lib/smartWorkoutParser.ts`)

**Purpose**: Parse workout text into structured exercises

**Note**: This logic is **server-side only** (calls AWS Bedrock). Android app should call `/api/ai/enhance-workout` endpoint instead of reimplementing.

**Android Implementation**:
```kotlin
// Don't port this - use API call instead
suspend fun enhanceWorkout(rawText: String): EnhanceWorkoutResponse {
    return spotBuddyApi.enhanceWorkout(
        EnhanceWorkoutRequest(rawText = rawText)
    )
}
```

#### D. Instagram Parser (`src/lib/igParser.ts`)

**Purpose**: Extract workout content from Instagram captions

**Note**: Also **server-side only**. Android should call `/api/instagram-fetch` endpoint.

---

### ✅ 3. API Endpoints (REST API Contract)

The Android app calls these Next.js API routes. You need to match the **exact request/response formats**.

#### Authentication

```kotlin
interface SpotBuddyApi {

    // POST /api/auth/login (Note: May use NextAuth.js OAuth flow instead)
    @POST("/api/auth/login")
    suspend fun login(@Body request: LoginRequest): LoginResponse

    data class LoginRequest(val email: String, val password: String)
    data class LoginResponse(
        val token: String,
        val user: UserDto
    )
}
```

**Note**: Web app uses **AWS Cognito + Google OAuth** via NextAuth.js. Android should use **AWS Cognito Android SDK** directly.

#### Workouts

```kotlin
// GET /api/workouts - List all user workouts
@GET("/api/workouts")
suspend fun getWorkouts(): WorkoutsResponse

data class WorkoutsResponse(
    val workouts: List<WorkoutDto>
)

// POST /api/workouts - Create workout
@POST("/api/workouts")
suspend fun createWorkout(@Body request: CreateWorkoutRequest): WorkoutResponse

data class CreateWorkoutRequest(
    val title: String,
    val description: String?,
    val exercises: List<ExerciseDto>,
    val content: String,
    val source: String,
    val type: String,
    val totalDuration: Int,
    val difficulty: String,
    val tags: List<String>
)

// GET /api/workouts/{id} - Get specific workout
@GET("/api/workouts/{id}")
suspend fun getWorkout(@Path("id") workoutId: String): WorkoutResponse

// PATCH /api/workouts/{id} - Update workout
@PATCH("/api/workouts/{id}")
suspend fun updateWorkout(
    @Path("id") workoutId: String,
    @Body updates: UpdateWorkoutRequest
): WorkoutResponse

// DELETE /api/workouts/{id}
@DELETE("/api/workouts/{id}")
suspend fun deleteWorkout(@Path("id") workoutId: String)

// PATCH /api/workouts/{id}/schedule - Schedule workout
@PATCH("/api/workouts/{id}/schedule")
suspend fun scheduleWorkout(
    @Path("id") workoutId: String,
    @Body request: ScheduleWorkoutRequest
): WorkoutResponse

data class ScheduleWorkoutRequest(
    val scheduledDate: String, // YYYY-MM-DD
    val status: WorkoutStatus = WorkoutStatus.SCHEDULED
)

// POST /api/workouts/{id}/complete - Mark completed
@POST("/api/workouts/{id}/complete")
suspend fun completeWorkout(
    @Path("id") workoutId: String,
    @Body request: CompleteWorkoutRequest
): WorkoutResponse

data class CompleteWorkoutRequest(
    val completedDate: String // YYYY-MM-DD
)

// GET /api/workouts/scheduled?date=YYYY-MM-DD
@GET("/api/workouts/scheduled")
suspend fun getScheduledWorkouts(
    @Query("date") date: String? = null
): ScheduledWorkoutsResponse
```

#### OCR

```kotlin
// POST /api/ocr - Process image with OCR
@POST("/api/ocr")
@Multipart
suspend fun processOCR(
    @Part image: MultipartBody.Part
): OCRResponse

data class OCRResponse(
    val text: String,
    val confidence: Double?
)
```

#### AI Enhancement

```kotlin
// POST /api/ai/enhance-workout
@POST("/api/ai/enhance-workout")
suspend fun enhanceWorkout(
    @Body request: EnhanceWorkoutRequest
): EnhanceWorkoutResponse

data class EnhanceWorkoutRequest(
    val rawText: String
)

data class EnhanceWorkoutResponse(
    val enhancedText: String,
    val suggestions: List<String>
)

// POST /api/ai/generate-workout
@POST("/api/ai/generate-workout")
suspend fun generateWorkout(
    @Body request: GenerateWorkoutRequest
): GenerateWorkoutResponse

data class GenerateWorkoutRequest(
    val prompt: String
)

data class GenerateWorkoutResponse(
    val workout: WorkoutDto,
    val cost: Double,
    val tokensUsed: Int
)
```

#### Training Profile

```kotlin
// GET /api/user/profile
@GET("/api/user/profile")
suspend fun getProfile(): ProfileResponse

// PUT /api/user/profile
@PUT("/api/user/profile")
suspend fun updateProfile(
    @Body profile: UpdateProfileRequest
): ProfileResponse

data class TrainingProfileDto(
    val personalRecords: Map<String, PREntry>,
    val goals: TrainingGoals,
    val preferences: TrainingPreferences,
    val experience: String,
    val constraints: TrainingConstraints
)
```

#### Body Metrics

```kotlin
// GET /api/body-metrics
@GET("/api/body-metrics")
suspend fun getBodyMetrics(): BodyMetricsResponse

// POST /api/body-metrics
@POST("/api/body-metrics")
suspend fun createBodyMetric(@Body metric: BodyMetricDto): BodyMetricResponse

// GET /api/body-metrics/{date}
@GET("/api/body-metrics/{date}")
suspend fun getBodyMetric(@Path("date") date: String): BodyMetricResponse

// PATCH /api/body-metrics/{date}
@PATCH("/api/body-metrics/{date}")
suspend fun updateBodyMetric(
    @Path("date") date: String,
    @Body updates: UpdateBodyMetricRequest
): BodyMetricResponse

// GET /api/body-metrics/latest
@GET("/api/body-metrics/latest")
suspend fun getLatestBodyMetric(): BodyMetricResponse
```

---

### ✅ 4. Feature Gating Logic

**TypeScript** (`src/lib/feature-gating.tsx`):
```typescript
// Subscription tier limits
const TIER_LIMITS = {
  free: {
    maxWorkouts: 15,
    ocrPerWeek: 1,
    historyDays: 30,
    aiEnhancementsPerMonth: 0,
    aiGenerationsPerMonth: 0,
  },
  starter: {
    maxWorkouts: Infinity,
    ocrPerWeek: 5,
    historyDays: Infinity,
    aiEnhancementsPerMonth: 10,
    aiGenerationsPerMonth: 0,
  },
  pro: {
    maxWorkouts: Infinity,
    ocrPerWeek: Infinity,
    historyDays: Infinity,
    aiEnhancementsPerMonth: 30,
    aiGenerationsPerMonth: 30,
  },
  elite: {
    maxWorkouts: Infinity,
    ocrPerWeek: Infinity,
    historyDays: Infinity,
    aiEnhancementsPerMonth: 100,
    aiGenerationsPerMonth: 100,
  },
};
```

**Kotlin**:
```kotlin
// File: app/src/main/java/com/spotbuddy/app/domain/model/TierLimits.kt
object TierLimits {

    data class Limits(
        val maxWorkouts: Int,
        val ocrPerWeek: Int,
        val historyDays: Int,
        val aiEnhancementsPerMonth: Int,
        val aiGenerationsPerMonth: Int
    )

    fun getLimits(tier: SubscriptionTier): Limits {
        return when (tier) {
            SubscriptionTier.FREE -> Limits(
                maxWorkouts = 15,
                ocrPerWeek = 1,
                historyDays = 30,
                aiEnhancementsPerMonth = 0,
                aiGenerationsPerMonth = 0
            )
            SubscriptionTier.STARTER -> Limits(
                maxWorkouts = Int.MAX_VALUE,
                ocrPerWeek = 5,
                historyDays = Int.MAX_VALUE,
                aiEnhancementsPerMonth = 10,
                aiGenerationsPerMonth = 0
            )
            SubscriptionTier.PRO -> Limits(
                maxWorkouts = Int.MAX_VALUE,
                ocrPerWeek = Int.MAX_VALUE,
                historyDays = Int.MAX_VALUE,
                aiEnhancementsPerMonth = 30,
                aiGenerationsPerMonth = 30
            )
            SubscriptionTier.ELITE -> Limits(
                maxWorkouts = Int.MAX_VALUE,
                ocrPerWeek = Int.MAX_VALUE,
                historyDays = Int.MAX_VALUE,
                aiEnhancementsPerMonth = 100,
                aiGenerationsPerMonth = 100
            )
        }
    }

    fun canUseFeature(user: User, feature: Feature): Boolean {
        val limits = getLimits(user.subscriptionTier)
        return when (feature) {
            Feature.ADD_WORKOUT -> user.workoutsSaved < limits.maxWorkouts
            Feature.OCR -> user.ocrQuotaUsed < limits.ocrPerWeek
            Feature.AI_ENHANCEMENT -> (user.aiRequestsUsed ?: 0) < limits.aiEnhancementsPerMonth
            Feature.AI_GENERATION -> (user.aiRequestsUsed ?: 0) < limits.aiGenerationsPerMonth
        }
    }

    enum class Feature {
        ADD_WORKOUT, OCR, AI_ENHANCEMENT, AI_GENERATION
    }
}
```

---

### ❌ 5. What You DON'T Need from Web App

These are web-specific and not relevant for Android:

- ❌ **Next.js pages** (`src/app/**/*page.tsx`) - Build native Compose UI instead
- ❌ **React components** (`src/components/**/*.tsx`) - Use Jetpack Compose
- ❌ **NextAuth.js** (`src/lib/auth-options.ts`) - Use AWS Cognito Android SDK
- ❌ **API route handlers** (`src/app/api/**/route.ts`) - Android calls these as a client
- ❌ **Tailwind CSS / styling** - Use Material Design 3 theming
- ❌ **Server-side AI logic** (`src/lib/ai/*`) - Call REST APIs instead

---

## Summary: What to Copy/Port

| Web App File | Android Equivalent | Action |
|--------------|-------------------|--------|
| `src/lib/dynamodb.ts` (types) | `data/remote/dto/*Dto.kt` | **Port data models** |
| `src/lib/pr-calculator.ts` | `domain/usecase/Calculate1RMUseCase.kt` | **Port algorithm** |
| `src/lib/exercise-history.ts` | `domain/usecase/ExtractExerciseHistoryUseCase.kt` | **Port logic** |
| `src/lib/feature-gating.tsx` | `domain/model/TierLimits.kt` | **Port business rules** |
| `src/app/api/**/route.ts` | `data/remote/api/SpotBuddyApi.kt` | **Match API contracts** |
| `src/lib/ai/*` | N/A | **Call REST APIs** (don't port) |
| `src/app/**/*page.tsx` | `ui/**/*Screen.kt` | **Rebuild UI in Compose** |

---

## Quick Start for Android Dev

1. **Start with data models**: Copy TypeScript interfaces → Kotlin data classes
2. **Set up Retrofit**: Create API interface matching Next.js routes
3. **Port business logic**: Rewrite PR calculator, exercise history in Kotlin
4. **Test API integration**: Verify Android can call web backend
5. **Build UI**: Create Compose screens (don't port React components)

**Reference Files**:
- See web app `/src/lib/dynamodb.ts` for complete data models
- See web app `/src/lib/pr-calculator.ts` for 1RM algorithms
- See web app `/src/lib/exercise-history.ts` for stats logic
- See web app API routes for endpoint specifications

---

**Last Updated**: October 25, 2025
**Web App Commit**: f12e6c7 (AI features complete)
