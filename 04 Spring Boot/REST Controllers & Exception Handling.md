# REST Controllers & Exception Handling

## REST Controller Annotations

- `@RestController` = `@Controller` + `@ResponseBody` — return values serialize straight to the response body instead of resolving a view name

| | `@PathVariable` | `@RequestParam` |
|---|---|---|
| Source | Part of the URL structure itself | Query string, tacked on |
| Omitted by client | No match → **404**, request never routes | `required=true` by default → **400**, never reaches method |
| Optional? | Not really — it's the route pattern | Only if `required=false` or `defaultValue(...)` set |

> [!danger] Trap
> Neither one just "becomes null" when missing. Both fail the request *before your method runs* unless explicitly declared optional.

**Wrapper vs primitive when actually optional (`required = false`):**

| Type | Value when omitted | Why |
|---|---|---|
| `Boolean` (wrapper) | `null` | Object reference, can hold null |
| `boolean` (primitive) | `false` (type's zero-value) | Primitives can't hold null — Spring falls back to default |

> [!danger] Trap
> `if (includeItems)` on a null `Boolean` → `NullPointerException` on auto-unboxing. Same check on primitive `boolean` can never NPE.

---

## Exception Handling — `@ControllerAdvice`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(f -> f.getField() + ": " + f.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse(message));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        return ResponseEntity.status(500).body(new ErrorResponse("Unexpected error"));
    }
}
```

- `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`, applies globally across every controller
- `@Valid @RequestBody` field validation failure → **`MethodArgumentNotValidException`** (not the same as a missing `@RequestParam`, which throws `MissingServletRequestParameterException`)

### Handler resolution rule
- Spring walks the thrown exception's class hierarchy and picks the **closest matching** `@ExceptionHandler` in the advice class — not declaration order
- An unlisted exception type (no exact/ancestor match anywhere in the advice class) **skips the advice entirely** — falls through to Spring Boot's default `BasicErrorController` → generic 500

> [!danger] Trap — narrow advice is a coverage gap, not a safety net
> A `@ControllerAdvice` with only specific handlers (`MethodArgumentNotValidException`, `EntityNotFoundException`, etc.) does **not** catch everything thrown in a controller. Anything not explicitly matched bypasses the class completely. Production code keeps a generic `@ExceptionHandler(Exception.class)` specifically as a deliberate catch-all.

**Mnemonic:** *Handlers are opt-in per exception type — unmatched types don't fall through your class, they fall through it entirely.*
