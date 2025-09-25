# AI Agent Guidelines

This document provides guidelines for AI agents working on this Roblox Farm Tycoon project. The codebase uses modern Luau practices with comprehensive type annotations and follows established patterns.

## Project Overview

This is a Roblox game built with:
- **Luau** with comprehensive type annotations
- **ReplicaService** for client-server state synchronization
- **ProfileService** for data persistence
- **Fusion** for reactive UI components
- **Custom packages** for common functionality (Maid, Promise, Signal, etc.)

## Luau Type System

### Basic Luau Types
Luau is a statically-typed language that extends Lua. Here are the fundamental types:

```luau
-- Primitive types
local name: string = "Player"
local age: number = 25
local isActive: boolean = true
local data: any = { anything = "goes here" }
local unknown: unknown = someValue -- Use when type is truly unknown

-- Collections
local items: { string } = { "apple", "banana" } -- Array of strings
local playerData: { [string]: any } = { coins = 100, level = 5 } -- Dictionary
local mixed: { string | number } = { "hello", 42 } -- Union array

-- Functions
local callback: (string, number) -> boolean = function(name, value) return true end
local voidFunction: () -> () = function() end

-- Optional types (can be nil)
local optionalString: string? = nil
local optionalNumber: number? = 42

-- Union types (one of several types)
local id: string | number = "player123"
local status: "active" | "inactive" | "pending" = "active"

-- Intersection types (must satisfy all types)
local player: Player & { Data: PlayerData } = getPlayer()
```

### Type Definitions
```luau
-- Define custom types
export type PlayerData = {
    coins: number,
    level: number,
    inventory: { ItemRecord },
    lastLogin: number?,
}

-- Generic types
export type Result<T> = {
    success: boolean,
    data: T?,
    error: string?,
}

-- Interface-like types
export type Service = {
    name: string,
    start: (self: Service) -> (),
    stop: (self: Service) -> (),
}
```

### Type Assertions & Guards
```luau
-- Type assertion (use sparingly)
local player = workspace.Player as Player

-- Type guard function
local function isPlayer(obj: any): obj is Player
    return typeof(obj) == "Instance" and obj:IsA("Player")
end

-- Runtime type checking
if typeof(value) == "table" then
    -- value is now typed as table
end
```

## Code Style & Standards

### Type Annotations
- **Always use type annotations** for function parameters, return values, and variables
- Use `export type` for type definitions that need to be shared across modules
- Prefer specific types over `any` - use `unknown` when the type is truly unknown
- Use union types (`|`) and intersection types (`&`) appropriately

```luau
-- Good
export type PlayerData = {
    coins: number,
    level: number,
    inventory: { ItemRecord },
}

local function processPlayer(player: Player, data: PlayerData): boolean
    -- implementation
end

-- Avoid
local function processPlayer(player, data)
    -- implementation
end
```

### Module Structure
- Use `export type` for types that other modules need
- Return `nil` from type-only modules (like `src/Shared/Types/init.luau`)
- Use descriptive class documentation with `@class` annotations
- Include parameter and return type documentation

```luau
--[=[
    @class ShopService

    Handles shop management, including stock for multiple shop types
]=]
local ShopService = {}

export type ShopData = {
    supportedItemTypes: { string },
    replica: ShopData.ShopReplicaServer,
}
```

### Error Handling
- Use `Promise` for asynchronous operations
- Handle errors gracefully with proper error messages
- Use `assert()` for critical failures that should stop execution
- Log errors with context for debugging

```luau
local function purchaseItem(player: Player, itemId: string): Promise<boolean>
    return Promise.new(function(resolve, reject)
        -- Validate input
        if not player or not itemId then
            reject("Invalid parameters")
            return
        end

        -- Process purchase
        local success = processPurchase(player, itemId)
        if success then
            resolve(true)
        else
            reject("Purchase failed")
        end
    end)
end
```

## Architecture Patterns

### Service Pattern
Services are singleton modules that handle specific game functionality:
- Located in `src/Server/Features/`
- Use `export type` for service-specific types
- Handle both client and server logic as appropriate
- Use ReplicaService for state synchronization

### Data Layer
- **Shared Data**: `src/Shared/Data/` - Configuration and static data
- **Types**: `src/Shared/Types/` - Type definitions
- **ProfileService**: Used for persistent player data
- **ReplicaService**: Used for real-time state synchronization

### Client-Server Communication
- Use **ReplicaService** for state synchronization
- Use **RemoteEvents** for one-time actions
- Use **RemoteFunctions** for request-response patterns
- Always validate data on the server side

## File Organization

### Directory Structure
```
src/
├── Client/           # Client-side code
│   ├── Features/     # Client feature modules
│   ├── UI/          # UI components and logic
│   └── Util/        # Client utilities
├── Server/          # Server-side code
│   ├── Features/    # Server feature modules
│   ├── Handlers/    # RemoteEvent/Function handlers
│   └── Util/        # Server utilities
└── Shared/          # Shared between client and server
    ├── Data/        # Configuration and static data
    ├── Types/       # Type definitions
    ├── Class/       # Shared classes
    └── Modules/     # Shared utility modules
```

### Naming Conventions
- **Services**: `XxxService.luau` (e.g., `ShopService.luau`)
- **Types**: `XxxTypes.luau` (e.g., `ItemTypes.luau`)
- **Data**: `XxxData.luau` (e.g., `ShopData.luau`)
- **Handlers**: `Xxx.luau` (e.g., `SeedShop.luau`)
- **Classes**: `Xxx.luau` (e.g., `Player.luau`)

## Common Patterns

### ReplicaService Usage
```luau
-- Server side
local replica = Replica.new({
    Name = "PlayerData",
    Data = playerData,
    Replication = "All",
})

-- Client side
local replica = ReplicaController.GetReplicaByClass("PlayerData")
local playerData = replica.Data
```

### ProfileService Integration
```luau
local ProfileService = require(ServerScriptService.Features.Core.ProfileService)

local function onPlayerAdded(player: Player)
    local profile = ProfileService.LoadProfileAsync(player.UserId)
    if profile then
        -- Handle successful profile load
    else
        -- Handle profile load failure
    end
end
```

### Fusion UI Components
```luau
local Fusion = require(ReplicatedStorage.Packages.Fusion)
local New = Fusion.New
local Children = Fusion.Children

local function createButton(text: string): GuiButton
    return New "TextButton" {
        Text = text,
        Size = UDim2.fromScale(0.2, 0.1),
        [Children] = {
            -- child components
        }
    }
end
```

## Development Tools

### Linting & Formatting
- **Selene**: Custom configuration in `seleneCustom.yml`
- **StyLua**: Code formatting (configured in `stylua.toml`)
- **Luau LSP**: Language server for type checking and IntelliSense

### Package Management
- **Wally**: Package manager for Luau packages
- **Aftman**: Tool version manager
- Custom packages in `Packages/` directory

## Best Practices

### Performance
- Use object pooling for frequently created/destroyed objects
- Cache expensive calculations
- Use `RunService.Heartbeat` for frame-based updates
- Minimize RemoteEvent calls

### Security
- Always validate data on the server
- Use proper RBAC (Role-Based Access Control)
- Sanitize user input
- Never trust client data

### Code Quality
- Write self-documenting code with clear variable names
- Use consistent indentation (tabs)
- Add comments for complex logic
- Keep functions focused and small
- Use early returns to reduce nesting

### Testing
- Test both client and server functionality
- Verify ReplicaService synchronization
- Test edge cases and error conditions
- Validate data persistence with ProfileService

## Common Gotchas

1. **ReplicaService**: Remember to call `ReplicaController.RequestData()` on the client
2. **ProfileService**: Profiles are automatically released when players leave
3. **RemoteEvents**: Use `FireAllClients()` sparingly for performance
4. **Types**: Use `export type` for types that need to be imported elsewhere
5. **Promises**: Always handle both resolve and reject cases

## Debugging

### Debug Configuration
```luau
local DEBUG_ENABLED = false -- Toggle for debug prints

local function debugPrint(message: string, ...)
    if DEBUG_ENABLED then
        print("[ServiceName DEBUG]", string.format(message, ...))
    end
end
```

### Common Debug Tools
- Use `print()` statements with service prefixes
- Check ReplicaService data with `ReplicaController.GetReplicasByClass()`
- Verify ProfileService data with `ProfileService.GetProfileStore()`
- Use Roblox Studio's Output window for debugging

## When Making Changes

1. **Understand the existing patterns** before making changes
2. **Maintain type safety** - add types for new functionality
3. **Test both client and server** behavior
4. **Update related documentation** if changing APIs
5. **Follow the established directory structure**
6. **Use existing utility functions** from the Packages directory
7. **Consider performance implications** of changes
8. **Ensure backward compatibility** when possible

## Package Dependencies & Documentation

### Core Packages
- **Promise**: [evaera/promise@4.0.0](https://eryn.io/roblox-lua-promise/api/Promise) - Promise implementation for asynchronous operations
- **Fusion**: [elttob/fusion@0.3.0](https://elttob.github.io/Fusion/) - Reactive UI framework
- **t**: [osyrisrblx/t@3.1.1](https://osyrisrblx.github.io/t/) - Type checking library
- **red**: [dig1t/red@2.2.7](https://dig1t.github.io/roblox-modules) - State management and actions

### Custom dig1t Packages
All custom packages are documented at: [https://dig1t.github.io/roblox-modules](https://dig1t.github.io/roblox-modules)

- **Animation**: [dig1t/animation@1.0.8](https://dig1t.github.io/roblox-modules) - Animation utilities
- **Badge**: [dig1t/badge@1.0.7](https://dig1t.github.io/roblox-modules) - Badge management
- **Cache**: [dig1t/cache@1.0.10](https://dig1t.github.io/roblox-modules) - Caching utilities
- **GamePass**: [dig1t/gamepass@1.0.9](https://dig1t.github.io/roblox-modules) - Game pass handling
- **Maid**: [dig1t/maid@1.1.2](https://dig1t.github.io/roblox-modules) - Cleanup management
- **Palette**: [dig1t/palette@1.0.1](https://dig1t.github.io/roblox-modules) - Color palette management
- **ProfileDB**: [dig1t/profiledb@1.0.8](https://dig1t.github.io/roblox-modules) - ProfileService wrapper
- **Ragdoll**: [dig1t/ragdoll@1.0.4](https://dig1t.github.io/roblox-modules) - Ragdoll physics
- **Replica**: [dig1t/replica@1.0.13](https://dig1t.github.io/roblox-modules) - ReplicaService wrapper
- **Signal**: [dig1t/signal@1.0.3](https://dig1t.github.io/roblox-modules) - Event system
- **State**: [dig1t/state@1.2.2](https://dig1t.github.io/roblox-modules) - State management
- **Trash**: [dig1t/trash@1.0.4](https://dig1t.github.io/roblox-modules) - Object disposal
- **Util**: [dig1t/util@1.0.19](https://dig1t.github.io/roblox-modules) - Utility functions

## Resources

### Official Documentation
- [Roblox Creator Documentation](https://github.com/Roblox/creator-docs) - **Primary reference for all Roblox development**
- [Luau Language Reference](https://luau-lang.org/) - Luau language documentation
- [Luau GitHub Repository](https://github.com/Roblox/luau) - Luau source code and development
- [Luau Type System Documentation](https://luau-lang.org/typecheck) - Comprehensive type system guide
- [Luau Syntax Documentation](https://luau-lang.org/syntax) - Language syntax reference
- [ReplicaService Documentation](https://madstudioroblox.github.io/ReplicaService/) - Client-server state synchronization
- [ProfileService Documentation](https://madstudioroblox.github.io/ProfileService/) - Data persistence

### Development Tools
- [Selene Documentation](https://kampfkarren.github.io/selene/) - Luau linter
- [StyLua Documentation](https://github.com/JohnnyMorganz/StyLua) - Code formatter
- [Luau LSP](https://github.com/JohnnyMorganz/luau-lsp) - Language server

## Important Notes

**Always reference the [Roblox Creator Documentation](https://github.com/Roblox/creator-docs) for:**
- Roblox API usage and best practices
- Service-specific documentation
- Platform-specific considerations
- Performance guidelines
- Security recommendations

Remember: This codebase prioritizes type safety, performance, and maintainability. When in doubt, follow existing patterns, consult the package documentation, and reference the official Roblox creator docs for platform-specific guidance.
