# SDL2 Translation Cheat Sheet (C to Pascal)

This cheat sheet helps to quickly translate often found C constructs in the
SDL2 package into Pascal according to the Code style guidelines of the
conversion project.

## Defines

C:

```
#define SDL_HAT_CENTERED    0x00
#define SDL_HAT_UP          0x01
#define SDL_HAT_RIGHT       0x02
#define SDL_HAT_DOWN        0x04
#define SDL_HAT_LEFT        0x08
#define SDL_HAT_RIGHTUP     (SDL_HAT_RIGHT|SDL_HAT_UP)
#define SDL_HAT_RIGHTDOWN   (SDL_HAT_RIGHT|SDL_HAT_DOWN)
#define SDL_HAT_LEFTUP      (SDL_HAT_LEFT|SDL_HAT_UP)
#define SDL_HAT_LEFTDOWN    (SDL_HAT_LEFT|SDL_HAT_DOWN)
```

Pascal:

```
const
  SDL_HAT_CENTERED  = $00;
  SDL_HAT_UP        = $01;
  SDL_HAT_RIGHT     = $02;
  SDL_HAT_DOWN      = $04;
  SDL_HAT_LEFT      = $08;
  SDL_HAT_RIGHTUP   = SDL_HAT_RIGHT or SDL_HAT_UP;
  SDL_HAT_RIGHTDOWN = SDL_HAT_RIGHT or SDL_HAT_DOWN;
  SDL_HAT_LEFTUP    = SDL_HAT_LEFT or SDL_HAT_UP;
  SDL_HAT_LEFTDOWN  = SDL_HAT_LEFT or SDL_HAT_DOWN;
```

## Enums

C:

```
typedef enum
{
   SDL_JOYSTICK_POWER_UNKNOWN = -1,
   SDL_JOYSTICK_POWER_EMPTY,   /* <= 5% */
   SDL_JOYSTICK_POWER_LOW,     /* <= 20% */
   SDL_JOYSTICK_POWER_MEDIUM,  /* <= 70% */
   SDL_JOYSTICK_POWER_FULL,    /* <= 100% */
   SDL_JOYSTICK_POWER_WIRED,
   SDL_JOYSTICK_POWER_MAX
} SDL_JoystickPowerLevel;
```

Pascal:

```
type
  TSDL_JoystickPowerLevel = type Integer;

const
  SDL_JOYSTICK_POWER_UNKNOWN = TSDL_JoystickPowerLevel(-1);
  SDL_JOYSTICK_POWER_EMPTY   = TSDL_JoystickPowerLevel(0);  {* <= 5% *}
  SDL_JOYSTICK_POWER_LOW     = TSDL_JoystickPowerLevel(1);  {* <= 20% *}
  SDL_JOYSTICK_POWER_MEDIUM  = TSDL_JoystickPowerLevel(2);  {* <= 70% *}
  SDL_JOYSTICK_POWER_FULL    = TSDL_JoystickPowerLevel(3);  {* <= 100% *}
  SDL_JOYSTICK_POWER_WIRED   = TSDL_JoystickPowerLevel(4);
  SDL_JOYSTICK_POWER_MAX     = TSDL_JoystickPowerLevel(5);
```

Hint 1: C enums start at 0 if no explicit value is set.

Hint 2: The type should be Word if only unsigned values are possible. Otherwise
it should be Integer.

Hint 3: Do not translate C enums to Pascal enums. C enums are handled like plain
integers which will make bitwise operations (e. g. in macros) possible
without typecasting.

## Structs

C:

```
typedef struct SDL_version
{
    Uint8 major;        /**< major version */
    Uint8 minor;        /**< minor version */
    Uint8 patch;        /**< update version */
} SDL_version;
```

Pascal:

```
type
  PSDL_Version = ^TSDL_Version;
  TSDL_Version = record
    major: cuint8    { major version }
    minor: cuint8    { minor version }
    patch: cuint8;   { update version }
  end;
```

Hint 1: If you have something like ```typedef struct name name```. the concrete
structure is probably opaque. You should translate it as follows, although
the best way to handle this is still not finally decided on. (see issue
[#63](https://github.com/PascalGameDevelopment/SDL2-for-Pascal/issues/63))

C:

```
typedef struct SDL_Window SDL_Window;
```

Pascal:

```
type
  PSDL_Window = ^TSDL_Window;
  TSDL_Window = record end;
```

## Unions

C:

```
typedef union {
    /** \brief A cutoff alpha value for binarization of the window shape's alpha channel. */
    Uint8 binarizationCutoff;
    SDL_Color colorKey;
} SDL_WindowShapeParams;
```

Pascal:

```
type
  PSDL_WindowShapeParams = ^TSDL_WindowShapeParams;
  TSDL_WindowShapeParams = record
  case cint of
    { A cutoff alpha value for binarization of the window shape's alpha channel. }
    0: (binarizationCutoff: cuint8);
    1: (colorKey: TSDL_ColorKey);
  end;
```

## Functions

C:

```
extern DECLSPEC void SDLCALL SDL_LockJoysticks(void);

extern DECLSPEC const char *SDLCALL SDL_JoystickNameForIndex(int device_index);
```

Pascal:

```
procedure SDL_LockJoysticks(); cdecl;
  external SDL_LibName {$IFDEF DELPHI} {$IFDEF MACOS} name '_SDL_LockJoysticks' {$ENDIF} {$ENDIF};

function SDL_JoystickNameForIndex(device_index: cint): PAnsiChar; cdecl;
  external SDL_LibName {$IFDEF DELPHI} {$IFDEF MACOS} name '_SDL_JoystickNameForIndex' {$ENDIF} {$ENDIF};
```

## C Macros

Macros are pre-processed constructs in C which have no analogue in Pascal.
Usually a C macro is translated as a Pascal function and implemented in SDL2.pas.

C:

```
#define SDL_VERSION_ATLEAST(X, Y, Z) \
    (SDL_COMPILEDVERSION >= SDL_VERSIONNUM(X, Y, Z))
```

Pascal:

_sdlversion.inc (declaration)_:
```
function SDL_VERSION_ATLEAST(X,Y,Z: cuint8): Boolean;
```

_sdl2.pas (implementation)_:
```
function SDL_VERSION_ATLEAST(X,Y,Z: cuint8): Boolean;
begin
  Result := SDL_COMPILEDVERSION >= SDL_VERSIONNUM(X,Y,Z);
end;
```
Hint: As can be seen from this example, the macro has no clearly defined
argument types and return value type. The types to be used for arguments and
return values depend on the context in which this macro is used. Here from the
context is known that X, Y and Z stand for the major version, minor version and
the patch level which are declared to be 8 bit unsigned integers. From the
context it is also clear, that the macro returns true or false, hence the
function should return a Boolean value. The context does not suggest to use,
e. g., TSDL_Bool here, although in a different context this could be the better
translation.

