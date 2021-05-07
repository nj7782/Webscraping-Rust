![Rust](https://img.shields.io/badge/-Rust-orange)

# Webscraping-Rust

In this post I'm going to write a Rust script that scrapes (web-scraping) information about the 10 most COVID affected countries with the details (total cases, total deaths, and total recovered) from worldometers.info to a CSV file?

## Scraping Ecosystem

I will be using the following libraries 

- [reqwest]

    > An easy and powerful Rust HTTP Client

- [scraper]

    > HTML parsing and querying with CSS selectors

- [select.rs]

    > A Rust library to extract useful data from HTML documents, suitable for web scraping
    
- [tokio::main]
 
   > A runtime for writing reliable network applications without compromising speed.

- [csv::Writer]
  
  > A CSV writer takes as input Rust values and writes those values in a valid CSV format as output.
   
The first script will perform a fairly basic task: grabbing all the covid data from the page. For this, we'll utilize `reqwest` and `scraper`. As you can see the syntax is fairly concise and straightforward.

```rust

extern crate reqwest;
extern crate scraper;

use scraper::{Selector, Html};
use csv::Writer;
use std::fs;
use std::fs::File;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {

    let resp = reqwest::get("https://www.worldometers.info/coronavirus/#countries").await?;
    assert!(resp.status().is_success());


    let body = resp.text().await?;

    let fragment = Html::parse_document(&body);

    let countries = Selector::parse("#main_table_countries_today tbody:nth-child(2) tr:not([style*=\"display: none\"]):not(.total_row_world)").unwrap();

    let mut count = 1;
    
 ```
    
   I want top 10 covid countries data to wrap up in a csv file. So, I will create a path and select the data which I require.
    
    
 ```rust
    let mut wtr = Writer::from_path("data.csv")?;

    wtr.write_record(&["#", "Country", "Total Cases", "Total Deaths", "Total Recovered"])?;


    for country in fragment.select(&countries) {

        let mut country_txt = country.text().collect::<Vec<_>>();

        country_txt.retain(|x| *x != "\n" && !x.starts_with("+"));

        wtr.write_record(&[country_txt[0], country_txt[1], country_txt[2], country_txt[3], country_txt[4]])?;


        if cnt >= 10 { break; }
        else {cnt += 1;}
    }


    wtr.flush()?;
    Ok(())
}
```

The end result of running this script is as follows:

![final](https://i.imgur.com/eNlN22v.png)

#,Country,Total Cases,Total Deaths,Total Recovered
1,USA,"32,475,043","581,542 ","25,043,463"
2,India,"15,321,089","180,550 ","13,108,582"
3,Brazil,"13,977,713","375,049 ","12,460,712"
4,France,"5,296,222","101,180 ","4,150,842"
5,Russia,"4,718,854","106,307 ","4,343,229"
6,UK,"4,390,783","127,274 ","4,156,135"
7,Turkey,"4,323,596","36,267 ","3,736,537"
8,Italy,"
 ","3,878,994","117,243 "
9,Spain,"3,428,354","77,102 ","3,144,353"
10,Germany,"3,164,447","80,774 ","2,803,600"

Thanks For Watching my repo:)
