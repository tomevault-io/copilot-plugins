## diet-whisperer

> FastAPI endpoints, request/response patterns, authentication, API documentation, or backend service logic.

# Diet Whisperer - API Patterns & FastAPI Conventions

**Use this rule when working on:** FastAPI endpoints, request/response patterns, authentication, API documentation, or backend service logic.

## FastAPI Application Structure

### Main Application (`apps/api/app/main.py`)
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import auth, diet_plans, meals, ai_agent

app = FastAPI(
    title="Diet Whisperer API",
    description="AI-powered diet planning assistant",
    version="1.0.0"
)

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://ai-dietary.herokuapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(auth.router, prefix="/auth", tags=["authentication"])
app.include_router(diet_plans.router, prefix="/diet-plans", tags=["diet-plans"])
app.include_router(meals.router, prefix="/meals", tags=["meals"])
app.include_router(ai_agent.router, prefix="/ai", tags=["ai-agent"])
```

## Router Organization

### File Structure
```
apps/api/app/routers/
├── __init__.py
├── auth.py          # Authentication endpoints
├── diet_plans.py    # Diet plan CRUD operations
├── meals.py         # Meal management with nutrition
├── ai_agent.py      # AI agent interactions
└── calendar.py      # Calendar feed generation
```

### Router Pattern
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session
from app.database import get_session
from app.models import User
from app.auth import get_current_user

router = APIRouter()

@router.get("/", response_model=List[DietPlanResponse])
async def get_diet_plans(
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session),
    skip: int = 0,
    limit: int = 100
):
    """Get user's diet plans with pagination"""
    # Implementation here
```

## Auto-Generation Service Patterns

### AutoGenerationService Class Structure
```python
# services/auto_generation_service.py
class AutoGenerationService:
    """Service for handling automatic meal generation for diet plans"""
    
    def __init__(self):
        self.agent = None
    
    def _get_agent(self):
        """Get or create the auto-generation agent"""
        if self.agent is None:
            self.agent = create_diet_auto_generation_agent()
        return self.agent
    
    async def run_auto_generation_for_all_plans(self) -> Dict[str, any]:
        """Run auto-generation for all eligible diet plans (scheduled task)"""
        # Implementation for batch processing
    
    async def generate_meals_for_plan(self, plan_id: int, days_to_generate: int) -> Dict[str, any]:
        """Generate meals for a specific diet plan (manual trigger endpoint)"""
        try:
            # Get diet plan information
            diet_plan = get_diet_plan_with_meals(plan_id)
            if not diet_plan:
                return {
                    "success": False,
                    "message": "Diet plan not found",
                    "error": "Diet plan not found"
                }
            
            # Create a plan dict in the format expected by _process_single_diet_plan
            plan_dict = {
                "id": plan_id,
                "user_id": diet_plan["user_id"],
                "auto_generate_days": days_to_generate,
                "last_auto_generation": diet_plan.get("last_auto_generation"),
                "user_email": "manual_trigger"  # Not used for single plan processing
            }
            
            # Use the existing processing logic
            result = await self._process_single_diet_plan(plan_dict)
            
            if result["success"]:
                if result["generated"]:
                    update_last_auto_generation(plan_id)
                    return {
                        "success": True,
                        "message": f"Successfully generated {result['days_generated']} days of meals",
                        "days_generated": result["days_generated"],
                        "start_date": result["start_date"]
                    }
                else:
                    return {
                        "success": False,
                        "message": result["reason"],
                        "error": result["reason"]
                    }
            else:
                return {
                    "success": False,
                    "message": "Generation failed",
                    "error": result["error"]
                }
        
        except Exception as e:
            logger.error(f"Error in generate_meals_for_plan: {str(e)}")
            return {
                "success": False,
                "message": f"Generation failed: {str(e)}",
                "error": str(e)
            }
    
    async def _process_single_diet_plan(self, plan: Dict) -> Dict[str, any]:
        """Process auto-generation for a single diet plan"""
        # Implementation for processing individual plans
```

### Auto-Generation Router Pattern
```python
# routers/diet_plans/auto_generation.py
@router.post("/{diet_plan_id}/auto-generate")
async def trigger_auto_generation(
    diet_plan_id: int, current_user: dict = Depends(get_current_user)
):
    """Manually trigger auto-generation for a diet plan"""
    
    # Verify user owns this diet plan
    diet_plan = get_diet_plan_with_meals(diet_plan_id)
    if not diet_plan:
        raise HTTPException(status_code=404, detail="Diet plan not found")
    
    if diet_plan["user_id"] != current_user["id"]:
        raise HTTPException(
            status_code=403, detail="Not authorized to modify this diet plan"
        )
    
    # Get auto-generation settings
    settings = get_auto_generation_settings(diet_plan_id)
    if not settings or not settings["auto_generate_enabled"]:
        raise HTTPException(
            status_code=400,
            detail="Auto-generation is not enabled for this diet plan",
        )
    
    # Use the auto-generation service
    try:
        from services.auto_generation_service import AutoGenerationService
        
        service = AutoGenerationService()
        result = await service.generate_meals_for_plan(
            diet_plan_id, settings["auto_generate_days"]
        )
        
        if result["success"]:
            return AutoGenerationResponse(
                success=True,
                message=result["message"],
                days_generated=result.get("days_generated", 0),
                start_date=result.get("start_date"),
            )
        else:
            return AutoGenerationResponse(
                success=False,
                message=result["message"],
                error=result.get("error"),
            )
    except ImportError:
        raise HTTPException(
            status_code=500,
            detail="Auto-generation service not available",
        )
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Auto-generation failed: {str(e)}",
        )
```

### Auto-Generation Response Models
```python
class AutoGenerationSettings(BaseModel):
    auto_generate_enabled: bool
    auto_generate_days: int = Field(ge=5, le=14)  # 5, 7, or 14 days

class AutoGenerationResponse(BaseModel):
    success: bool
    message: str
    days_generated: Optional[int] = None
    start_date: Optional[str] = None
    error: Optional[str] = None
```

### Auto-Generation Error Handling
```python
# Custom exception handler for auto-generation
@app.exception_handler(Exception)
async def auto_generation_exception_handler(request: Request, exc: Exception):
    """Handle auto-generation specific errors"""
    
    error_message = str(exc)
    
    if "AutoGenerationService" in error_message and "has no attribute" in error_message:
        return JSONResponse(
            status_code=500,
            content={
                "detail": "Auto-generation service error",
                "message": "The auto-generation service is missing required methods",
                "suggestion": "Please check the service implementation"
            }
        )
    
    if "generate_meals_for_plan" in error_message:
        return JSONResponse(
            status_code=500,
            content={
                "detail": "Auto-generation method error",
                "message": error_message,
                "suggestion": "Ensure the AutoGenerationService has the generate_meals_for_plan method"
            }
        )
    
    # Standard error response
    return JSONResponse(
        status_code=500,
        content={"detail": error_message}
    )
```

## Request/Response Patterns

### Pydantic Models

#### Request Models with Auto-Nutrition
```python
class MealCreate(BaseModel):
    day_date: str = Field(regex=r"^\d{4}-\d{2}-\d{2}$")
    timeframe: str = Field(regex=r"^\d{2}:\d{2}:\d{2}$")
    meal_type: str = Field(regex=r"^(breakfast|lunch|dinner|snack)$")
    food_description: str = Field(
        min_length=1, 
        max_length=1000,
        description="Detailed meal description - be specific about portions and cooking methods for accurate AI nutrition analysis"
    )
    
    # Nutritional information (optional - auto-calculated if not provided)
    calories: Optional[int] = Field(ge=0, le=5000, description="Calories per serving (auto-calculated if not provided)")
    protein_g: Optional[float] = Field(ge=0, le=200, description="Protein in grams (auto-calculated if not provided)")
    carbs_g: Optional[float] = Field(ge=0, le=500, description="Carbohydrates in grams (auto-calculated if not provided)")
    fat_g: Optional[float] = Field(ge=0, le=200, description="Fat in grams (auto-calculated if not provided)")
    fiber_g: Optional[float] = Field(ge=0, le=50, description="Fiber in grams (auto-calculated if not provided)")

class MealUpdate(BaseModel):
    timeframe: Optional[str] = Field(regex=r"^\d{2}:\d{2}:\d{2}$")
    meal_type: Optional[str] = Field(regex=r"^(breakfast|lunch|dinner|snack)$")
    food_description: Optional[str] = Field(min_length=1, max_length=1000)
    
    # Optional nutrition updates
    calories: Optional[int] = Field(ge=0, le=5000)
    protein_g: Optional[float] = Field(ge=0, le=200)
    carbs_g: Optional[float] = Field(ge=0, le=500)
    fat_g: Optional[float] = Field(ge=0, le=200)
    fiber_g: Optional[float] = Field(ge=0, le=50)

class DietPlanCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    description: Optional[str] = Field(max_length=1000)
    start_date: date
    duration_days: int = Field(ge=1, le=30)
```

#### Response Models with Nutrition
```python
class MealResponse(BaseModel):
    id: int
    meal_type: str
    timeframe: Optional[str]
    food_description: str
    
    # Nutritional information
    calories: Optional[int]
    protein_g: Optional[float]
    carbs_g: Optional[float]
    fat_g: Optional[float]
    fiber_g: Optional[float]
    
    created_at: datetime

class DietDayResponse(BaseModel):
    id: int
    day_date: str
    meals: List[MealResponse]
    
    # Computed nutrition totals (calculated client-side)
    @computed_field
    @property
    def nutrition_totals(self) -> dict:
        """Calculate daily nutrition totals"""
        return {
            "calories": sum(meal.calories or 0 for meal in self.meals),
            "protein_g": sum(meal.protein_g or 0 for meal in self.meals),
            "carbs_g": sum(meal.carbs_g or 0 for meal in self.meals),
            "fat_g": sum(meal.fat_g or 0 for meal in self.meals),
            "fiber_g": sum(meal.fiber_g or 0 for meal in self.meals),
            "meal_count": len(self.meals)
        }

class DietPlanResponse(BaseModel):
    id: int
    title: str
    description: Optional[str]
    created_at: datetime
    diet_days: List[DietDayResponse]
```

### Endpoint Patterns

#### Meal Management with Nutrition
```python
# CREATE MEAL WITH AUTO-NUTRITION ANALYSIS
@router.post("/{diet_plan_id}/days/{day_date}/meals", response_model=MealResponse, status_code=status.HTTP_201_CREATED)
async def create_meal(
    diet_plan_id: int,
    day_date: str,
    meal_data: MealCreate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """
    Create a new meal with automatic nutrition analysis.
    
    If nutrition values are not provided, the system will automatically analyze
    the food_description using LangChain + OpenAI with user context for personalized estimates.
    """
    try:
        # Verify user owns the diet plan
        diet_plan = get_diet_plan_with_meals(diet_plan_id)
        if not diet_plan or diet_plan["user_id"] != current_user.id:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Diet plan not found or access denied"
            )
        
        # Create meal with auto-nutrition analysis
        meal_id = add_meal_to_diet_day(
            diet_plan_id=diet_plan_id,
            day_date=day_date,
            timeframe=meal_data.timeframe,
            meal_type=meal_data.meal_type,
            food_description=meal_data.food_description,
            calories=meal_data.calories,  # Optional - will be auto-calculated if None
            protein_g=meal_data.protein_g,  # Optional - will be auto-calculated if None
            carbs_g=meal_data.carbs_g,  # Optional - will be auto-calculated if None
            fat_g=meal_data.fat_g,  # Optional - will be auto-calculated if None
            fiber_g=meal_data.fiber_g,  # Optional - will be auto-calculated if None
            user_id=current_user.id,  # For personalized nutrition analysis
            auto_analyze_nutrition=True,  # Enable LangChain nutrition analysis
        )
        
        # Return created meal with nutrition data
        return {
            "message": "Meal created successfully",
            "meal_id": meal_id,
            "nutrition_analyzed": True
        }
        
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Failed to create meal: {str(e)}"
        )

# UPDATE MEAL WITH NUTRITION
@router.put("/{meal_id}", response_model=MealResponse)
async def update_meal_endpoint(
    meal_id: int,
    meal_update: MealUpdate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Update meal including nutritional information"""
    
    # Verify meal ownership
    meal_info = get_meal_with_diet_plan_info(meal_id)
    if not meal_info or meal_info["user_id"] != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Meal not found"
        )
    
    # Update meal with only provided fields
    success = update_meal(
        meal_id=meal_id,
        timeframe=meal_update.timeframe,
        meal_type=meal_update.meal_type,
        food_description=meal_update.food_description,
        calories=meal_update.calories,
        protein_g=meal_update.protein_g,
        carbs_g=meal_update.carbs_g,
        fat_g=meal_update.fat_g,
        fiber_g=meal_update.fiber_g,
    )
    
    if not success:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to update meal"
        )
    
    # Return updated meal
    updated_meal = get_meal_by_id(meal_id)
    return updated_meal

# GET MEAL WITH NUTRITION
@router.get("/{meal_id}", response_model=MealResponse)
async def get_meal(
    meal_id: int,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Get meal with nutritional information"""
    
    meal_info = get_meal_with_diet_plan_info(meal_id)
    if not meal_info or meal_info["user_id"] != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Meal not found"
        )
    
    return meal_info
```

#### Diet Plan CRUD with Nutrition Data
```python
# CREATE DIET PLAN
@router.post("/", response_model=DietPlanResponse, status_code=status.HTTP_201_CREATED)
async def create_diet_plan(
    plan_data: DietPlanCreate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Create a new diet plan"""
    try:
        # Validation
        if plan_data.start_date < date.today():
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Start date cannot be in the past"
            )
        
        # Create plan structure only (meals added via AI agent)
        diet_plan_id = create_diet_plan_only(
            user_id=current_user.id,
            title=plan_data.title,
            description=plan_data.description or ""
        )
        
        # Return created plan
        plan = get_diet_plan_with_meals(diet_plan_id)
        return plan
        
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )

# GET DIET PLAN WITH NUTRITION
@router.get("/{plan_id}", response_model=DietPlanResponse)
async def get_diet_plan(
    plan_id: int,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Get diet plan with all meals and nutrition data"""
    
    plan = get_diet_plan_with_meals(plan_id)
    if not plan or plan["user_id"] != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Diet plan not found"
        )
    
    return plan
```

## Authentication Patterns

### JWT Token Authentication
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    session: Session = Depends(get_session)
) -> User:
    """Get current authenticated user"""
    try:
        payload = jwt.decode(
            credentials.credentials,
            SECRET_KEY,
            algorithms=[ALGORITHM]
        )
        email: str = payload.get("sub")
        if email is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid authentication credentials"
            )
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )
    
    user = get_user_by_email(email)
    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    return user
```

### Login Endpoint
```python
@router.post("/login", response_model=TokenResponse)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    session: Session = Depends(get_session)
):
    """Authenticate user and return JWT token"""
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    access_token = create_access_token(data={"sub": user["email"]})
    return TokenResponse(access_token=access_token, token_type="bearer")
```

## AI Agent Integration Patterns

### Streaming Responses with Nutrition
```python
from fastapi.responses import StreamingResponse
from app.ai_agent import DietPlannerAgent

@router.post("/chat/{diet_plan_id}")
async def chat_with_diet_plan(
    diet_plan_id: int,
    message: ChatMessage,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Stream AI agent response for diet plan conversation"""
    
    # Verify diet plan ownership
    plan = get_diet_plan_with_meals(diet_plan_id)
    if not plan or plan["user_id"] != current_user.id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Diet plan not found"
        )
    
    # Stream response with nutrition updates
    async def generate_response():
        async for chunk in stream_agent_response(
            agent=DietPlannerAgent(),
            thread_id=str(diet_plan_id),
            message=message.content,
            diet_plan_id=diet_plan_id
        ):
            yield f"data: {json.dumps(chunk)}\n\n"
    
    return StreamingResponse(
        generate_response(),
        media_type="text/plain",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

### Diet Plan Generation Endpoint
```python
@router.post("/generate-plan", response_model=DietPlanResponse)
async def generate_diet_plan(
    plan_request: DietPlanGenerateRequest,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
):
    """Generate diet plan using AI agent with nutrition"""
    
    try:
        # Create initial plan structure
        diet_plan_id = create_diet_plan_only(
            user_id=current_user.id,
            title=plan_request.title,
            description="AI-generated diet plan"
        )
        
        # Generate meals with nutrition via AI agent
        agent = DietPlannerAgent()
        result = await agent.generate_complete_plan(
            diet_plan_id=diet_plan_id,
            user_preferences=plan_request.preferences,
            duration_days=plan_request.duration_days
        )
        
        if not result["success"]:
            raise HTTPException(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                detail=result.get("error", "Failed to generate plan")
            )
        
        # Return complete plan with nutrition data
        plan = get_diet_plan_with_meals(diet_plan_id)
        return plan
        
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=str(e)
        )
```

## Error Handling Patterns

### Nutrition Validation Errors
```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def nutrition_validation_exception_handler(request: Request, exc: RequestValidationError):
    """Handle nutrition validation errors with helpful messages"""
    
    nutrition_errors = []
    for error in exc.errors():
        field = error.get("loc", [])[-1] if error.get("loc") else "unknown"
        
        if field in ["calories", "protein_g", "carbs_g", "fat_g", "fiber_g"]:
            nutrition_errors.append({
                "field": field,
                "message": f"Invalid {field}: {error.get('msg', 'validation error')}",
                "input": error.get("input")
            })
    
    if nutrition_errors:
        return JSONResponse(
            status_code=422,
            content={
                "detail": "Nutrition validation error",
                "nutrition_errors": nutrition_errors,
                "help": "Ensure all nutrition values are positive numbers within reasonable ranges"
            }
        )
    
    # Standard validation error response
    return JSONResponse(
        status_code=422,
        content={
            "detail": "Validation error",
            "errors": exc.errors()
        }
    )
```

### Custom Exception Handlers
```python
@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    """Handle nutrition calculation errors"""
    error_message = str(exc)
    
    if "nutrition" in error_message.lower():
        return JSONResponse(
            status_code=400,
            content={
                "detail": "Nutrition calculation error",
                "message": error_message,
                "suggestion": "Please verify nutritional values and try again"
            }
        )
    
    return JSONResponse(
        status_code=400,
        content={"detail": error_message}
    )
```

## Calendar Integration Patterns

### iCal Feed Generation with Timezone Support
```python
@router.get("/{diet_plan_id}/ics")
async def get_diet_calendar_ics(diet_plan_id: int, timezone: str = "UTC"):
    """
    Get .ics calendar feed for a specific diet plan with timezone support
    This endpoint can be subscribed to in calendar applications

    :param diet_plan_id: Diet plan ID
    :param timezone: IANA timezone identifier (e.g., 'America/New_York', 'Europe/London')
    :return: ICS calendar file with proper timezone handling
    """
    # Find the diet plan
    diet_plan = get_diet_plan_with_meals(diet_plan_id)
    if not diet_plan:
        raise HTTPException(status_code=404, detail="Diet plan not found")

    # Import timezone handling
    from datetime import timezone as dt_timezone
    from zoneinfo import ZoneInfo
    
    # Validate and get timezone
    try:
        if timezone != "UTC":
            tz = ZoneInfo(timezone)
        else:
            tz = dt_timezone.utc
    except Exception:
        # Fallback to UTC if invalid timezone
        tz = dt_timezone.utc
        timezone = "UTC"

    # Generate ICS content with timezone
    ics_content = [
        "BEGIN:VCALENDAR",
        "VERSION:2.0",
        "PRODID:-//DietCraft AI//Diet Calendar//EN",
        "CALSCALE:GREGORIAN",
        "METHOD:PUBLISH",
        f'X-WR-CALNAME:DietCraft - {diet_plan["title"]}',
        f'X-WR-CALDESC:{diet_plan["description"]}',
        f"X-WR-TIMEZONE:{timezone}",
    ]

    # Add timezone definition if not UTC
    if timezone != "UTC":
        ics_content.extend([
            "BEGIN:VTIMEZONE",
            f"TZID:{timezone}",
            "END:VTIMEZONE"
        ])

    def format_date_for_ics_with_tz(dt: datetime, use_timezone: str) -> str:
        if use_timezone == "UTC":
            return dt.strftime("%Y%m%dT%H%M%SZ")
        else:
            return dt.strftime("%Y%m%dT%H%M%S")

    for day in diet_plan["days"]:
        # Process meals
        for meal in day["meals"]:
            try:
                # Parse the date and time
                day_date = datetime.fromisoformat(day["day_date"]).date()
                time_parts = meal["timeframe"].split(":")
                hours, minutes = int(time_parts[0]), int(time_parts[1])

                # Create start time in specified timezone
                start_time = datetime.combine(day_date, time(hours, minutes))
                if timezone != "UTC":
                    start_time = start_time.replace(tzinfo=tz)
                
                end_time = start_time + timedelta(minutes=30)

                # Format dates with timezone
                dtstart = format_date_for_ics_with_tz(start_time, timezone)
                dtend = format_date_for_ics_with_tz(end_time, timezone)
                
                # Add timezone parameter if not UTC
                if timezone != "UTC":
                    dtstart_line = f"DTSTART;TZID={timezone}:{dtstart}"
                    dtend_line = f"DTEND;TZID={timezone}:{dtend}"
                else:
                    dtstart_line = f"DTSTART:{dtstart}"
                    dtend_line = f"DTEND:{dtend}"

                # Add nutrition to description
                nutrition_info = []
                if meal.get('calories'):
                    nutrition_info.append(f"Calories: {meal['calories']}")
                if meal.get('protein_g'):
                    nutrition_info.append(f"Protein: {meal['protein_g']}g")
                if meal.get('carbs_g'):
                    nutrition_info.append(f"Carbs: {meal['carbs_g']}g")
                
                description = meal['food_description']
                if nutrition_info:
                    description += f"\n\nNutrition: {', '.join(nutrition_info)}"

                # Use meal title as summary, fallback to food_description
                meal_title = meal.get("title")
                if not meal_title or meal_title.strip() == "":
                    # Fallback: use first part of food_description as title
                    meal_title = meal["food_description"][:50] + ("..." if len(meal["food_description"]) > 50 else "")
                
                # Use detailed food_description as the event description
                meal_description = meal["food_description"]
                if nutrition_info:
                    meal_description += f"\n\nNutrition: {', '.join(nutrition_info)}"

                ics_content.extend([
                    "BEGIN:VEVENT",
                    f'UID:{diet_plan_id}-{day["day_date"]}-meal-{meal["meal_type"]}-{abs(hash(meal["food_description"]))}@dietcraft.ai',
                    dtstart_line,
                    dtend_line,
                    f'SUMMARY:{meal_title}',
                    f'DESCRIPTION:{meal_description}',
                    f'CATEGORIES:DIET,MEAL,{meal["meal_type"].upper()}',
                    "END:VEVENT",
                ])
            except Exception as e:
                # Skip problematic events but continue processing
                continue

        # Process sports activities
        for sport in day.get("sports", []):
            try:
                # Parse the date and time
                day_date = datetime.fromisoformat(day["day_date"]).date()
                time_parts = sport["timeframe"].split(":")
                hours, minutes = int(time_parts[0]), int(time_parts[1])

                # Create start time in specified timezone
                start_time = datetime.combine(day_date, time(hours, minutes))
                if timezone != "UTC":
                    start_time = start_time.replace(tzinfo=tz)

                # End time based on duration or default to 60 minutes
                duration_minutes = sport.get("duration_minutes", 60)
                end_time = start_time + timedelta(minutes=duration_minutes)

                # Format dates with timezone
                dtstart = format_date_for_ics_with_tz(start_time, timezone)
                dtend = format_date_for_ics_with_tz(end_time, timezone)
                
                # Add timezone parameter if not UTC
                if timezone != "UTC":
                    dtstart_line = f"DTSTART;TZID={timezone}:{dtstart}"
                    dtend_line = f"DTEND;TZID={timezone}:{dtend}"
                else:
                    dtstart_line = f"DTSTART:{dtstart}"
                    dtend_line = f"DTEND:{dtend}"

                # Build description with sport details
                description_parts = [sport["description"]]
                if sport.get("intensity"):
                    description_parts.append(f"Intensity: {sport['intensity']}")
                if sport.get("calories_burned"):
                    description_parts.append(f"Calories burned: {sport['calories_burned']}")
                if sport.get("notes"):
                    description_parts.append(f"Notes: {sport['notes']}")
                
                description = "\n".join(description_parts)

                ics_content.extend([
                    "BEGIN:VEVENT",
                    f'UID:{diet_plan_id}-{day["day_date"]}-sport-{sport["sport_type"]}-{abs(hash(sport["description"]))}@dietcraft.ai',
                    dtstart_line,
                    dtend_line,
                    f'SUMMARY:{sport["sport_type"].title()}: {sport["description"]}',
                    f'DESCRIPTION:{description}',
                    f'CATEGORIES:SPORT,FITNESS,{sport["sport_type"].upper()}',
                    "END:VEVENT",
                ])
            except Exception as e:
                # Skip problematic events but continue processing
                continue

    ics_content.append("END:VCALENDAR")

    return Response(
        content="\r\n".join(ics_content),
        media_type="text/calendar",
        headers={
            "Content-Disposition": f'attachment; filename="diet-plan-{diet_plan_id}.ics"',
            "Cache-Control": "no-cache, no-store, must-revalidate",
            "Pragma": "no-cache",
            "Expires": "0",
        },
    )
```

### Frontend Calendar Integration with Timezone
```typescript
// Frontend timezone detection and calendar URL generation
const getBrowserTimezone = (): string => {
    try {
        return Intl.DateTimeFormat().resolvedOptions().timeZone;
    } catch (error) {
        console.warn('Failed to get browser timezone, falling back to UTC:', error);
        return 'UTC';
    }
};

const getSubscriptionUrl = (dietPlanId: number): string => {
    const timezone = getBrowserTimezone();
    return `${config.getApiUrl()}/api/diet-calendar/${dietPlanId}/ics?timezone=${encodeURIComponent(timezone)}`;
};

// Frontend ICS export with timezone support
const exportToICS = (dietPlan: DietPlan) => {
    const timezone = getBrowserTimezone();
    
    const icsContent = [
        'BEGIN:VCALENDAR',
        'VERSION:2.0',
        'PRODID:-//DietCraft AI//Diet Calendar//EN',
        'CALSCALE:GREGORIAN',
        'METHOD:PUBLISH',
        `X-WR-CALNAME:DietCraft - ${dietPlan.title}`,
        `X-WR-TIMEZONE:${timezone}`,
    ];

    // Add timezone definition if not UTC
    if (timezone !== 'UTC') {
        icsContent.push(
            'BEGIN:VTIMEZONE',
            `TZID:${timezone}`,
            'END:VTIMEZONE'
        );
    }

    dietPlan.days.forEach(day => {
        // Add meals
        day.meals.forEach(meal => {
            const date = new Date(day.day_date);
            const [hours, minutes] = meal.timeframe.split(':').map(Number);
            
            const startTime = new Date(date);
            startTime.setHours(hours, minutes, 0, 0);
            const endTime = new Date(startTime);
            endTime.setMinutes(endTime.getMinutes() + 30);

            const formatDateForICS = (date: Date): string => {
                if (timezone === 'UTC') {
                    return date.toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z';
                } else {
                    const year = date.getFullYear();
                    const month = String(date.getMonth() + 1).padStart(2, '0');
                    const day = String(date.getDate()).padStart(2, '0');
                    const hours = String(date.getHours()).padStart(2, '0');
                    const minutes = String(date.getMinutes()).padStart(2, '0');
                    const seconds = String(date.getSeconds()).padStart(2, '0');
                    return `${year}${month}${day}T${hours}${minutes}${seconds}`;
                }
            };

            const dtstart = formatDateForICS(startTime);
            const dtend = formatDateForICS(endTime);
            
            const dtstartLine = timezone !== 'UTC' 
                ? `DTSTART;TZID=${timezone}:${dtstart}`
                : `DTSTART:${dtstart}`;
            const dtendLine = timezone !== 'UTC'
                ? `DTEND;TZID=${timezone}:${dtend}`
                : `DTEND:${dtend}`;

            icsContent.push(
                'BEGIN:VEVENT',
                `UID:${dietPlan.id}-${day.day_date}-meal-${meal.meal_type}-${Date.now()}@dietcraft.ai`,
                dtstartLine,
                dtendLine,
                `SUMMARY:${meal.meal_type.charAt(0).toUpperCase() + meal.meal_type.slice(1)}: ${meal.food_description}`,
                `DESCRIPTION:${meal.food_description}`,
                `CATEGORIES:DIET,MEAL,${meal.meal_type.toUpperCase()}`,
                'END:VEVENT'
            );
        });

        // Add sports activities if they exist
        if (day.sports) {
            day.sports.forEach(sport => {
                const date = new Date(day.day_date);
                const [hours, minutes] = sport.timeframe.split(':').map(Number);
                
                const startTime = new Date(date);
                startTime.setHours(hours, minutes, 0, 0);
                const endTime = new Date(startTime);
                const durationMinutes = sport.duration_minutes || 60;
                endTime.setMinutes(endTime.getMinutes() + durationMinutes);

                // Build description with sport details
                const descriptionParts = [sport.description];
                if (sport.intensity) {
                    descriptionParts.push(`Intensity: ${sport.intensity}`);
                }
                if (sport.calories_burned) {
                    descriptionParts.push(`Calories burned: ${sport.calories_burned}`);
                }
                if (sport.notes) {
                    descriptionParts.push(`Notes: ${sport.notes}`);
                }

                const dtstart = formatDateForICS(startTime);
                const dtend = formatDateForICS(endTime);
                
                const dtstartLine = timezone !== 'UTC' 
                    ? `DTSTART;TZID=${timezone}:${dtstart}`
                    : `DTSTART:${dtstart}`;
                const dtendLine = timezone !== 'UTC'
                    ? `DTEND;TZID=${timezone}:${dtend}`
                    : `DTEND:${dtend}`;

                icsContent.push(
                    'BEGIN:VEVENT',
                    `UID:${dietPlan.id}-${day.day_date}-sport-${sport.sport_type}-${Date.now()}@dietcraft.ai`,
                    dtstartLine,
                    dtendLine,
                    `SUMMARY:${sport.sport_type.charAt(0).toUpperCase() + sport.sport_type.slice(1)}: ${sport.description}`,
                    `DESCRIPTION:${descriptionParts.join('\n')}`,
                    `CATEGORIES:SPORT,FITNESS,${sport.sport_type.toUpperCase()}`,
                    'END:VEVENT'
                );
            });
        }
    });

    icsContent.push('END:VCALENDAR');
    const finalIcsContent = icsContent.join('\r\n');

    const blob = new Blob([finalIcsContent], { type: 'text/calendar' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `diet-plan-${dietPlan.id}.ics`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
};
```

## Testing Patterns

### Test Client Setup with Nutrition
```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_meal_with_nutrition():
    """Test creating a meal with complete nutrition data"""
    response = client.post(
        "/meals/",
        json={
            "day_date": "2024-01-01",
            "timeframe": "08:00:00",
            "meal_type": "breakfast",
            "food_description": "Oatmeal with berries and nuts",
            "calories": 350,
            "protein_g": 12.0,
            "carbs_g": 45.0,
            "fat_g": 8.0,
            "fiber_g": 6.0
        },
        headers={"Authorization": f"Bearer {test_token}"}
    )
    assert response.status_code == 201
    
    meal_data = response.json()
    assert meal_data["calories"] == 350
    assert meal_data["protein_g"] == 12.0
    assert meal_data["food_description"] == "Oatmeal with berries and nuts"

def test_update_meal_nutrition():
    """Test updating meal nutrition data"""
    # Create meal first
    create_response = client.post("/meals/", json=test_meal_data, headers=auth_headers)
    meal_id = create_response.json()["id"]
    
    # Update nutrition
    update_response = client.put(
        f"/meals/{meal_id}",
        json={
            "calories": 400,
            "protein_g": 15.0
        },
        headers=auth_headers
    )
    assert update_response.status_code == 200
    
    updated_meal = update_response.json()
    assert updated_meal["calories"] == 400
    assert updated_meal["protein_g"] == 15.0
    # Other fields should remain unchanged
    assert updated_meal["food_description"] == test_meal_data["food_description"]

def test_diet_plan_nutrition_totals():
    """Test that diet plan response includes nutrition totals"""
    response = client.get(
        f"/diet-plans/{test_plan_id}",
        headers=auth_headers
    )
    assert response.status_code == 200
    
    plan_data = response.json()
    for day in plan_data["diet_days"]:
        # Check that nutrition totals are calculated
        assert "nutrition_totals" in day
        assert "calories" in day["nutrition_totals"]
        assert "protein_g" in day["nutrition_totals"]
        assert day["nutrition_totals"]["meal_count"] == len(day["meals"])
```

## Background Processing Patterns

### Simple Background Diet Plan Generation
```python
# Router: /api/diet-plans-simple
from fastapi import BackgroundTasks

@router.post("/generate", response_model=SimpleDietPlanResponse)
async def create_diet_plan_simple(
    request: SimpleDietPlanRequest,
    background_tasks: BackgroundTasks,
    current_user: dict = Depends(get_current_user),
) -> SimpleDietPlanResponse:
    """
    Start async diet plan generation using FastAPI BackgroundTasks
    Returns immediately with plan_id, processes in background
    """
    # Create initial plan record with 'pending' status
    plan_id = create_diet_plan_only(
        user_id=current_user["id"],
        title=request.title,
        description=request.description,
        status="pending",
    )
    
    # Start background generation
            result = BackgroundDietService.start_diet_plan_generation(
            background_tasks=background_tasks,
            user_id=current_user["id"],
            prompt=request.prompt,
            duration_days=request.duration_days,
            plan_id=plan_id,
        )
    
    return SimpleDietPlanResponse(**result)

@router.get("/{plan_id}/status")
async def get_diet_plan_status(
    plan_id: int, current_user: dict = Depends(get_current_user)
) -> Dict[str, Any]:
    """
    Poll generation status and progress
    Returns: status, progress, meal_count, is_completed
    """
    plan = get_diet_plan_with_meals(plan_id)
    status = plan.get("status", "unknown")
    meal_count = sum(len(day.get("meals", [])) for day in plan.get("days", []))
    
    # Calculate progress based on status and meals created
    if status == "pending":
        progress = 0
    elif status == "generating":
        expected_meals = 3 * len(plan.get("days", []))
        progress = min(50 + (meal_count / max(expected_meals, 1)) * 40, 90)
    elif status == "completed":
        progress = 100
    else:
        progress = 0
    
    return {
        "plan_id": plan_id,
        "status": status,
        "progress": int(progress),
        "meal_count": meal_count,
        "is_completed": status == "completed",
    }

@router.get("/{plan_id}/result")
async def get_diet_plan_result(plan_id: int, current_user: dict = Depends(get_current_user)):
    """Get completed diet plan or status message"""
    plan = get_diet_plan_with_meals(plan_id)
    status = plan.get("status", "unknown")
    
    if status == "completed":
        return {
            "status": "completed",
            "message": "Diet plan completed successfully!",
            "progress": 100,
            "diet_plan": plan,
        }
    else:
        return {"status": status, "message": f"Diet plan is {status}"}
```

### Request Models for Background Processing
```python
class SimpleDietPlanRequest(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    description: str = Field(max_length=1000)
    prompt: str = Field(min_length=10, max_length=2000)
    duration_days: int = Field(ge=1, le=30)

class SimpleDietPlanResponse(BaseModel):
    success: bool
    plan_id: int
    status: str  # "pending", "generating", "completed", "failed"
    message: str
    estimated_time_minutes: int
```

### Time Estimation Formula
```python
# In BackgroundDietService.start_diet_plan_generation()
estimated_seconds = 20 + (duration_days * 6)  # o4-mini model: 20s + 6s/day

estimated_minutes = estimated_seconds / 60.0
```

---

**Key API Principles:**
1. **Include nutrition in meal endpoints** - All meal operations support nutrition data
2. **Client-side nutrition calculation** - API returns raw meal data, frontend calculates totals
3. **Validate nutrition parameters** - Ensure positive values within reasonable ranges
4. **Background processing with FastAPI BackgroundTasks** - No external dependencies for async processing
5. **Status polling for long operations** - Real-time progress updates via polling endpoints
6. **Comprehensive error handling** - Specific error messages for nutrition validation
7. **Document nutrition fields** - Clear OpenAPI documentation for nutrition parameters
8. **Test nutrition workflows** - Comprehensive tests for nutrition CRUD operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/federicoBetti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
