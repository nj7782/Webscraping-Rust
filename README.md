![Rust](https://img.shields.io/badge/-Rust-orange)

# Webscraping-Rust

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
   
The first script will perform a fairly basic task: grabbing all links from the page. For this, we'll utilize `reqwest` and `select.rs`. As you can see the syntax is fairly concise and straightforward.

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
