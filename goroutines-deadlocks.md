

To avoid deadlock and race conditions when dealing with shared resources like a buffer in Go, you can use channels and proper synchronization techniques. Here's an example solution using channels and a select statement to handle communication between the goroutines:
```go
package main

import (
	"fmt"
	"time"
)

const bufferSize = 10

func main() {
	// Create a buffered channel to represent the shared buffer
	buffer := make(chan int, bufferSize)

	// Use channels to signal termination to goroutines
	done := make(chan struct{})

	// Initialize counters for readers and writers
	M := 5
	N := 3

	// Start M reader goroutines
	for i := 0; i < M; i++ {
		go reader(buffer, done, &M)
	}

	// Start N writer goroutines
	for i := 0; i < N; i++ {
		go writer(buffer, done, &N)
	}

	// Let the goroutines run for a while (for demonstration purposes)
	time.Sleep(5 * time.Second)

	// Signal termination to all goroutines
	close(done)

	// Wait for all goroutines to finish
	for M > 0 || N > 0 {
		select {
		case <-time.After(100 * time.Millisecond):
			// Give goroutines some time to finish
		}
	}

	fmt.Println("Program completed.")
}

func reader(buffer chan int, done chan struct{}, counter *int) {
	for {
		select {
		case <-done:
			// Terminate the goroutine
			return
		case value := <-buffer:
			// Process the value read from the buffer
			fmt.Println("Read:", value)
		}
	}
}

func writer(buffer chan int, done chan struct{}, counter *int) {
	for {
		select {
		case <-done:
			// Terminate the goroutine
			return
		default:
			// Write a value to the buffer
			value := 42
			buffer <- value
			fmt.Println("Write:", value)
			time.Sleep(100 * time.Millisecond)
		}
	}
}
```
This example uses two channels: buffer for communication between readers and writers, and done to signal termination to all goroutines. The select statement is used to handle multiple channels and avoid race conditions.

Note that the program includes a simple termination mechanism using the done channel and waits for the goroutines to finish before exiting. Adjust the values of M and N according to your specific requirements.