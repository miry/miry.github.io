---
url: https://dev.to/miry/1-billion-row-challenge-napkin-math-1mld
canonical_url: https://dev.to/miry/1-billion-row-challenge-napkin-math-1mld
title: 1 Billion Row Challenge Napkin Math
slug: 1-billion-row-challenge-napkin-math-1mld
description: Based on a real-world problem, this article uses a system design approach
  to calculate the input and output sizes for processing 1 billion rows of weather
  station data.
tags:
- programming
- systemdesign
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2024-07-04-1-billion-row-challenge-napkin-math-1mld-cover_image-https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Firmci5qzdlhrwm3283cp.jpg)

# 1 Billion Row Challenge Napkin Math


> Photo by <a href="https://unsplash.com/@clintnaik?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Clinton Naik</a> on <a href="https://unsplash.com/photos/lightnings-during-nighttime-NcTQ602gKLI?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

## Problem Overview

You need to write a program that processes 1 billion rows of weather station data to calculate the minimum, mean, and maximum temperatures for each station. This data is stored in a text file with over 10 GB of data, which presents significant computational challenges[^1].

This problem is an excellent exercise in data structure optimization and performance engineering. It offers a chance to explore various techniques across different programming languages to achieve efficient processing while adhering to constraints.

### Key Challenges

- **Handling Large Data Volumes:** Efficiently process over 10 GB of data without exhausting system memory.
- **Optimizing Data Structures:** Use appropriate data structures to store and compute temperature statistics.
- **Performance Considerations:** Implement algorithms that can handle the input size within a reasonable time frame.

## Napkin Math of Input and Output Data

Are there always 10GB?

We can try to calculate the size of the input with more precision. While there is a draft calculation of the `input.txt` file, let's calculate it ourselves.

Each row consists of a weather station name and a temperature value, separated by a semicolon. Example:

```
Hamburg;12.0
Bulawayo;8.9
Palembang;38.8
Hamburg;34.2
St. John's;15.2
Cracow;12.6
...
```

### Official Constraints

Here are summary of constraints from the task description.

- **Weather Station Name:**
  - Maximum length: 100 bytes
  - Minimum length: 1 character
- **Temperature Value:**
  - Range: -99.9 to 99.9 (always with one fractional digit)
  - Maximum length: 5 bytes (e.g., "-99.9")

These constraints are over-optimistic and cover more real-life cases with a lot of capacity.

### Nature of the Name

The input data consists of real-time entries where the first part of the line, separated by a semicolon (`;`), is the weather station name. Typically, we can assume this is a city name. To estimate the longest and average name lengths, consider the following:

- **Longest City Names:**
  - In the USA, the longest city name is 28 characters, and the shortest is 13 characters.[^2]
  - Globally, the top 10 cities with the longest names can have lengths up to 58 characters.[^3]

For practical purposes, we will use an estimated average length of 15 bytes for the city names. This conservative estimate helps in initial calculations, although the official constraint allows up to 100 bytes.

### Nature of the Temperature

The temperature value is predictable within certain ranges. Considering the Earth's recorded extremes:
* **Minimum Temperature:** -89.2°C (Antarctica).[^4]
* **Maximum Temperature:** 56.7°C (Death Valley, USA).[^5]

These values translate to string representations:
* The string representation of "-89.2" uses a maximum of 5 bytes.
* Temperatures between -9.9 and -0.1 use 4 bytes.
* Between 0.0 and 9.9 use 3 bytes.
* From 10.0 and up use 4 bytes.

Given that extreme cold is rarer than moderate and hot temperatures, we'll assume an average temperature string length of 4 bytes.

### Calculation Input

![Image description](/assets/2024-07-04-1-billion-row-challenge-napkin-math-1mld-9r7eo0kcmc9a4nm9z1ah.jpg)

> Photo by <a href="https://unsplash.com/@frankbouffard?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Francis Bouffard</a> on <a href="https://unsplash.com/photos/balck-corded-headphones-HADKIO0EFXQ?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  

Let's go wild and use just the official constraints, then compare with optimistic estimations.

#### Maximum File Size

**Per row**

100 bytes (station name) + 1 byte (semicolon) + 5 bytes (temperature) + 1 byte (newline) = 107 bytes

**Total**

1,000,000,000 rows * 107 bytes/row = 107,000,000,000 bytes ≈ 99.6 GiB

This is a lot. With these limitations, the resulting file should be extremely large. It would be interesting to see the results of the participants processing a 100 GiB file.

#### Estimated Optimistic File Size

**Per row**

15 bytes (station name) + 1 byte (semicolon) + 4 bytes (temperature) + 1 byte (newline) = 21 bytes

**Total**

1,000,000,000 rows * 21 bytes/row = 21,000,000,000 bytes ≈ 19.5 GiB

Oops, a 20 GiB file. It is twice as large as mentioned in the details. Even if we use 13 bytes for the city and 3 bytes for the weather, it would be a 17 GiB file.

### Calculation Output

Let's estimate the size of the output data:

For each weather station, the output format is: `StationName=min/mean/max`, where:
* StationName: up to 100 bytes
* min: 5 bytes (e.g., "-99.9")
* mean: 5 bytes (e.g., "-99.9" or "25.4")
* max: 5 bytes (e.g., "99.9")
* Additional characters: 1 byte for `=`, 2 bytes for `/`, and 2 bytes for `, ` (comma and space separating entries)

Thus, for each station:

**Total per station**: 100 bytes (station name) + 1 byte (=) + 5 bytes (min) + 1 byte (/) + 5 bytes (mean) + 1 byte (/) + 5 bytes (max) + 2 bytes (, ) = 120 bytes

With up to 10,000 unique stations, the total output size is:

**Total for all stations**: 120 bytes/station * 10,000 stations = 1,200,000 bytes ≈ 1.1 MiB

Note that braces `{...}` are not included in this calculation, as they would only be in the final row as a comma `, ` (comma and space separating entries).

## Summary

In summary, processing an estimated 19 GiB input file to generate a 1.1 MiB output file represents a significant data compression. This reflects the efficiency of summarizing temperature data per weather station, reducing the large dataset to meaningful statistics while adhering to given constraints. This exercise underscores the importance of efficient data processing and storage in handling large datasets.

![Image description](/assets/2024-07-04-1-billion-row-challenge-napkin-math-1mld-h4ubvzjajj8k0pgm6amz.png)

> That’s all folks!

[^1]: [1 Billion Row Challenge](https://1brc.dev/)
[^2]: [Precisely](https://customer.precisely.com/s/article/Maximum-length-of-a-City-name-according-to-the-USPS-for-use-with-CODE-1-Plus-and-Finalist?language=en_US)
[^3]: [Largest.org](https://largest.org/geography/city-names/)
[^4]: [Wikipedia - Lowest temperature recorded on Earth](https://en.wikipedia.org/wiki/Lowest_temperature_recorded_on_Earth)
[^5]: [Wikipedia - Highest temperature recorded on Earth](https://en.wikipedia.org/wiki/Highest_temperature_recorded_on_Earth)



