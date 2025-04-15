

1. Conditional Statements: Making Decisions

These statements allow your design to take different paths depending on whether a certain condition is true or false.

if Statement: The most basic conditional. It executes a block of code only if the specified condition is true.

Code snippet

if (reset) begin
    count = 0;
end
Here, the count will only be reset to 0 if the reset signal is high (true).

if-else Statement: This extends the if statement by providing an alternative block of code to execute if the initial condition is false.

Code snippet

if (data_valid) begin
    output_data = input_data;
end else begin
    output_data = 'bz; // High impedance if data is not valid
end
If data_valid is true, the input data is passed to the output; otherwise, the output goes to a high-impedance state.

if-else if-else Statement: This allows you to check multiple conditions sequentially. The first condition that evaluates to true will have its corresponding block executed. The final else block (if present) acts as a catch-all if none of the preceding conditions are met.

Code snippet

if (opcode == 2'b00) begin
    // Perform addition
    result = a + b;
end else if (opcode == 2'b01) begin
    // Perform subtraction
    result = a - b;
end else if (opcode == 2'b10) begin
    // Perform multiplication
    result = a * b;
end else begin
    // Invalid opcode
    result = 'bx; // Unknown value
end
This example shows how different operations can be performed based on the value of the opcode.

Nesting if Statements: You can embed if statements within other if or else blocks to create more complex decision-making structures. However, be mindful of excessive nesting, as it can make your code harder to read and debug.

Code snippet

if (enable) begin
    if (mode == 1'b1) begin
        // Perform operation in mode 1
    end else begin
        // Perform operation in mode 0
    end
end
2. Case Statements: Multi-Way Branching

When you need to compare an expression against several constant values, the case statement provides a cleaner and more readable alternative to a long chain of if-else if statements.

case Statement: Evaluates an expression (the "case expression") and compares its value against a series of "case items." When a match is found, the corresponding block of code is executed.

Code snippet

case (state)
    IDLE: begin
        next_state = FETCH;
        // Perform idle state actions
    end
    FETCH: begin
        next_state = DECODE;
        // Perform fetch state actions
    end
    DECODE: begin
        next_state = EXECUTE;
        // Perform decode state actions
    end
    default: begin
        next_state = IDLE;
        // Handle unexpected state
    end
endcase
Here, the state variable is compared against IDLE, FETCH, and DECODE. The default case handles any value of state that doesn't match the explicit cases.

casex Statement: Similar to case, but it treats x (unknown) and z (high impedance) values as "don't care" conditions in the case items. This can be useful when you want to match based on only certain bits of an expression.

Code snippet

casez (address) // Treats z as don't care
    4'b1???: begin
        // Address in the range 8-15
    end
    4'b01??: begin
        // Address in the range 4-7
    end
    default: begin
        // Other addresses
    end
endcase
The ? in casez acts as a wildcard, matching either 0, 1, x, or z. casex treats both x and z as wildcards.

casez Statement: Similar to casex, but it treats only z (high impedance) values as "don't care" conditions in the case items.

unique case, priority case: These are variations that enforce uniqueness or priority in the matching process.

unique case: Ensures that at most one case item matches the case expression. If multiple items match, it's an error.
priority case: Executes the first matching case item in the order they are listed, even if subsequent items could also match. This is important for implementing priority logic.
Code snippet

unique case (instruction)
    ADD: result = a + b;
    SUB: result = a - b;
    // ... other instructions
    default: $error("Invalid instruction");
endcase

priority case (interrupt_level)
    HIGH: handle_high_priority_interrupt();
    MEDIUM: handle_medium_priority_interrupt();
    LOW: handle_low_priority_interrupt();
endcase
3. Looping Statements: Repetitive Execution

Looping statements allow you to execute a block of code multiple times, which is essential for tasks like iterating through arrays, performing repetitive calculations, or simulating time.

for Loop: Executes a block of code a fixed number of times. It consists of three parts:

Initialization: Executed once at the beginning of the loop. Typically used to initialize a loop counter.
Condition: Evaluated before each iteration. The loop continues as long as the condition is true.
Increment/Decrement: Executed at the end of each iteration, usually to update the loop counter.
Code snippet

for (integer i = 0; i < 8; i++) begin
    memory[i] = 0; // Initialize the first 8 memory locations to 0
end
while Loop: Executes a block of code as long as a specified condition remains true. The condition is evaluated before each iteration.   

Code snippet

while (!fifo_empty) begin
    data_out = read_fifo();
    // Process the data
end
do-while Loop: Similar to the while loop, but the condition is evaluated after each iteration. This means the loop body will always execute at least once.

Code snippet

do begin
    data_out = read_fifo();
    // Process the data
end while (!fifo_empty);
foreach Loop: Provides a convenient way to iterate over the elements of an array.

Code snippet

logic [7:0] data_array [0:3];

foreach (data_array[i]) begin
    $display("Element [%0d]: %h", i, data_array[i]);
end
The foreach loop automatically iterates through all the indices of the data_array.

Important Considerations and Best Practices:

Blocking vs. Non-Blocking Assignments: Control flow statements often contain assignments. Remember the crucial difference between blocking (=) and non-blocking (<=) assignments, especially within always blocks. Blocking assignments execute sequentially, while non-blocking assignments are scheduled to update at the end of the current time step. Use non-blocking assignments for sequential logic (e.g., flip-flops) and blocking assignments for combinational logic.

Sensitivity Lists: When using control flow within always blocks, ensure your sensitivity list correctly captures all the signals that can affect the outcome. For combinational logic, use @(*). For sequential logic, use @(posedge clock) or @(posedge clock or negedge reset), for example.

Synthesis: Keep in mind that the control flow statements you write will be translated into hardware. Complex or infinite loops might not be synthesizable or could lead to unexpected hardware behavior. Ensure your control flow leads to well-defined hardware structures.

Readability and Maintainability: Use indentation and clear naming conventions to make your control flow structures easy to understand and maintain. Avoid overly complex nested structures if possible.

Simulation vs. Synthesis: While SystemVerilog is used for both simulation and synthesis, some constructs might behave differently in each domain. Always simulate your design thoroughly to verify its behavior before synthesis.





Code snippet

module traffic_light_controller (
    input logic clk,
    input logic rst,
    output logic red,
    output logic yellow,
    output logic green
);

    typedef enum logic [1:0] { RED_STATE, YELLOW_STATE, GREEN_STATE } state_t;
    state_t current_state, next_state;
    logic [27:0] timer; // A simple timer (adjust bit width as needed for desired duration)

    parameter integer RED_DURATION   = 50000000; // Example: 50 million clock cycles
    parameter integer YELLOW_DURATION = 5000000;  // Example: 5 million clock cycles
    parameter integer GREEN_DURATION  = 100000000; // Example: 100 million clock cycles

    // State transition logic (sequential)
    always_ff @(posedge clk or posedge rst) begin
        if (rst) begin
            current_state <= RED_STATE;
            timer <= 0;
        end else begin
            current_state <= next_state;
            if (next_state == current_state) begin
                if (timer < get_duration(current_state) - 1) begin
                    timer <= timer + 1;
                end else begin
                    timer <= 0; // Reset timer
                end
            end
        end
    end

    // Next state logic (combinational)
    always_comb begin
        next_state = current_state; // Default: stay in the current state
        case (current_state)
            RED_STATE:   if (timer == get_duration(RED_STATE) - 1)   next_state = GREEN_STATE;
            YELLOW_STATE: if (timer == get_duration(YELLOW_STATE) - 1) next_state = RED_STATE;
            GREEN_STATE:  if (timer == get_duration(GREEN_STATE) - 1)  next_state = YELLOW_STATE;
            default:      next_state = RED_STATE; // Should not happen, but good for safety
        endcase
    end

    // Output logic (combinational)
    always_comb begin
        red   = (current_state == RED_STATE);
        yellow = (current_state == YELLOW_STATE);
        green = (current_state == GREEN_STATE);
    end

    // Function to get the duration for the current state
    function integer get_duration (input state_t current);
        case (current)
            RED_STATE: return RED_DURATION;
            YELLOW_STATE: return YELLOW_DURATION;
            GREEN_STATE: return GREEN_DURATION;
            default: return 0;
        endcase
    endfunction

endmodule
In this example:

We use an enum (state_t) to define the different states of our traffic light: RED, YELLOW, and GREEN. This makes the code more readable than using magic numbers.
We have a state register (current_state) that holds the current state and a next-state register (next_state) to determine the transition.
An always_ff block with a clock and reset handles the sequential logic, updating the current_state on the rising edge of the clock or when the reset is asserted. It also includes a simple timer to keep track of how long the light has been in the current state.
An always_comb block defines the combinational logic for determining the next_state based on the current_state and whether the timer has reached the duration for the current state. We use a case statement here for clear multi-way branching based on the current_state.
Another always_comb block sets the output signals (red, yellow, green) based on the current_state.
A function called get_duration uses a case statement to return the appropriate duration for each state, making the timing parameters easily configurable.
This example demonstrates how if, else, case statements, and sequential logic (always_ff) work together to create a functional digital circuit. The traffic light cycles through its states based on the defined durations.





