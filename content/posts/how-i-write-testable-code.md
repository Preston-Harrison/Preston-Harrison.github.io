+++
title = 'How I write more testable code'
date = 2023-10-29T10:47:58-07:00
draft = false
+++

Early in my development journey, my functions often combined data fetching with data 
manipulation. This approach made testing cumbersome, as it required mocking
external IO operations. I've pivoted to two main strategies that have simplified my
testing process: separation of concerns and dependency injection. Let's dive
into my refined approach, complete with concrete Rust examples.

## The Old Approach: Merging Fetching with Manipulation

In my initial approach, functions typically resembled:
```rust
fn fetch_and_process_data(url: &str) -> Result<String, Error> {
    let data = http_client::get(url)?; // Fetching data
    let processed_data = data.chars().rev().collect(); // Manipulating data
    Ok(processed_data)
}
```

Testing `fetch_and_process_data` involved mocking the HTTP call, which is inefficient
and difficult.

## Separation of Concerns
The first method I embraced involved separating data fetching functions from those handling data manipulation.

```rust
fn fetch_data(url: &str) -> Result<String, Error> {
    http_client::get(url)
}

fn process_data(data: &str) -> String {
    data.chars().rev().collect()
}
```

With this separation, `process_data` became straightforward to test since it was devoid of any external IO.

## Dependency Injection
Sometimes, it isn't always feasable to seperate fetching the data from processing it. In that case, dependency injection
is a good solution. Dependency injection involves passing in the fetcher as a function argument, which makes mocking
the fetcher very easy for testing.

```rust
fn real_fetcher(url: &str) -> Result<String, Error> {
    http_client::get(url)
}

fn fetch_and_process_data<F>(url: &str, fetcher: fn(&str) -> Result<String, Error>) -> Result<String, Error> {
    let data = fetcher(url);
    let processed_data = data.chars().rev().collect();
    Ok(processed_data)
}

// Non-test case
fn main() {
    let _ = fetch_and_process_data("https://example.com", real_fetcher);
}

// Test case
#[cfg(test)]
mod tests {
    use super::*;

    fn mock_fetcher(_url: &str) -> Result<String, Error> {
        Ok("mocked data".to_string())
    }

    #[test]
    fn test_process_data_with_fn() {
        let processed = process_data_with_fn("http://example.com", mock_fetcher).unwrap();
        assert_eq!(processed, "atad dekcom".to_string());
    }
}
```

## Wrapping up

By implementing the separation of concerns, I made my functions more accessible 
to unit testing. Data manipulation functions, when isolated, become particularly 
easy to test due to the absence of external IO operations.

However, in scenarios where it's challenging to separate data fetching from 
data manipulation, dependency injection serves as a powerful alternative. By 
passing the fetcher function as an argument, we gain the flexibility to easily 
replace it with mock functions during testing, making our codebase more
adaptable and test-friendly.

Both these strategies have immensely improved the maintainability and test 
coverage of my projects.
