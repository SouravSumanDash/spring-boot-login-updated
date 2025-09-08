# spring-boot-login-updated


# Debug Task – Spring Boot Login API

## Objective

You are provided with a basic Spring Boot application containing a login endpoint at `/api/auth/login`. The endpoint is intended to authenticate users based on email and password. However, all login attempts currently result in a `401 Unauthorized` response, even when valid credentials are used.

Your task is to:

- Identify and resolve the issue(s) preventing successful authentication.
- Ensure that valid login attempts return a `200 OK` response along with a dummy JWT token.
- Ensure invalid login attempts return a proper `401 Unauthorized` response.
- Optionally improve the overall code quality, structure, or error handling where appropriate.

## Task Instructions

1. Clone or extract the provided project and open it in your preferred IDE.
2. Run the application using the standard Spring Boot method.
3. Attempt a login request using tools such as Postman or curl.
4. Diagnose why the authentication flow is not working as expected.
5. Fix the issues and test for the following:
   - Valid credentials return HTTP 200 with a sample JWT token.
   - Invalid credentials return HTTP 401 Unauthorized.
6. Keep the JWT logic simple—return a static token or placeholder string.

## Sample Credentials

Use the following sample user credentials (you may seed this user manually in memory or via data.sql):

Email: test@example.com  
Password: password123

Note: Ensure that the password is stored in a way that aligns with how Spring Security expects to verify it (e.g., password encoding if applicable).

## Evaluation Criteria

- Ability to identify and resolve issues related to authentication and configuration.
- Understanding of Spring Security concepts and correct usage.
- Clarity, readability, and maintainability of code changes.
- Appropriate use of HTTP status codes and request/response patterns.
- Optional: Suggestions or implementation of code improvements.

## Submission Instructions

- Submit the modified project as a ZIP file or via a public Git repository link.
- Include a brief section in this README file under **"Fix Summary"** explaining:
  - What issues were identified
  - What changes were made to fix them
  - Any improvements or recommendations made beyond the fix

## Fix Summary (to be filled by the candidate)

- Identified Issues:
  - ...

- Fixes Applied:
  - ...

- Improvements or Suggestions (optional):
  - ...


--------------------------------------------------------------------------------------------------------------------------------------
Solution:


## Fix Summary (to be filled by the candidate)


1. Identified Issues

From the logs and code you provided:

1. Authentication failed because Spring Security couldn’t find the user:

i)  Failed to find user 'NONE_PROVIDED' indicates the UsernamePasswordAuthenticationToken was constructed with a null or empty username.

ii) Authentication was failing because Spring Security was only checking username and ignored email.

iii) The login request is sending email/username, but the AuthenticationManager expects a single “username” field.

2. Password not encoded properly:

i) You need to store passwords as BCrypt hashes in your database.
ii)Passwords must be BCrypt hashed to match Spring Security’s encoder.

iii) Plain text passwords will fail Spring Security’s matches() check.

3. User table constraints:

i) H2 DB inserts failed because username cannot be null and the identifier column didn’t exist.

ii) Your data.sql should match your entity columns exactly (username, email, password).

4. Current login endpoint only accepts username:

i)  Your LoginRequest probably has username and password fields.

ii) If you want login via email or username, CustomerUserDetailsService must handle both.

--------------------------------------------------------------------------------------------------------------------------------------

2. Fixes Applied

1. Update LoginRequest to include either email or username:

public class LoginRequest {
    private String identifier; // email or username
    private String password;
    // getters and setters
}


2. Update AuthController to use identifier:

@PostMapping("/login")
public ResponseEntity<String> login(@RequestBody LoginRequest request) {
    Authentication auth = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(request.getIdentifier(), request.getPassword())
    );
    // Return a dummy JWT token
    return ResponseEntity.ok("{\"token\": \"dummy-jwt-token\"}");
}


3. Ensure CustomerUserDetailsService checks both email and username:

@Override
public UserDetails loadUserByUsername(String input) throws UsernameNotFoundException {
    return (input.contains("@") 
            ? userRepository.findByEmail(input)
            : userRepository.findByUsername(input))
        .map(user -> new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(),
                Collections.singleton(new SimpleGrantedAuthority("ROLE_USER"))
        ))
        .orElseThrow(() -> new UsernameNotFoundException("User not found with: " + input));
}


4. Seed H2 database properly (data.sql):

INSERT INTO users (username, email, password) VALUES
('testuser', 'test@example.com', '$2a$10$WY1bisoNopPsYGe8zvbg.uZ30s2gWVSIJ.K6tCXWM0DrJaj9.8KuC');


5. Configure PasswordEncoder bean:

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}


6. Test login using Postman:

Endpoint: POST http://localhost:8080/api/auth/login

Body (JSON):

{
  "identifier": "test@example.com",
  "password": "password123"
}

--------------------------------------------------------------------------------------------------------------------------------------
Output:-

Expected Response:

{"token": "dummy-jwt-token"}


Invalid credentials will return 401 Unauthorized.

3. Improvements or Suggestions

i)  Use JWT library (like jjwt) for real token generation instead of a static token.

ii)  Add exception handling with @ControllerAdvice for better error messages.

iii) Consider using DTOs for all API responses instead of plain strings.

iv) Add unit tests for login success and failure scenarios.







