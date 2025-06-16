# IMDb-web-scraper
This Jupyter Notebook features two URL web-scraping examples using Cinemagoer and BeautifulSoup Python modules: gathering movie lists from the Criterion Channel, and then determining the International Movie Database (IMDb) rating of those movies. 

### Description of program <a class="anchor" id="description"></a>

#### Function of program <a class="anchor" id="function_of_program"></a>
This program determines the rating of movies, particularly groups of movies listed on the Criterion Channel (https://www.criterionchannel.com/), with ratings provided by the International Movie Database (IMDb - https://www.imdb.com/).

#### Reason for developing <a class="anchor" id="reason_for_developing"></a>
I have enjoyed watching movies, especially film noir movies that are above a certain caliber, such as at least an approximate 7.0 out of 10.0 IMDb rating. I've found it tedious to always reference a movie's rating individually. This app will take a listing of movies featured on the Criterion Channel, or provided to it as a list of movies and their year of release and seek their rating. 

#### How it works <a class="anchor" id="how_it_works"></a>
The program first accepts a URL that links to a listing of movies, e.g., *https://www.criterionchannel.com/british-noir.* The function **criterion_list(url)** returnes a lising of movies under the British noir category with the year of release, in this case these movies: 

    Green for Danger (1946)
    Odd Man Out (1947)
    Obsession (1949)
    The Small Back Room (1949)
    The Woman in Question (1950)
    Hell Drivers (1957)
    Time Without Pity (1957)
    All Night Long (1962)

It does so by parsing the HTML page with BeautifulSoup. The movie title and year of release are found in *div-tagged* blocks of HTML code, with class attributes *tooltip background-white*: 

    blocks = soup.find_all('div', attrs = {'class':'tooltip background-white'})

A returned block of code with a movie title and year of release is shown:

    <div class="tooltip background-white" id="collection-tooltip-455466">
     <h3 class="tooltip-item-title site-font-primary-family">
      <strong>
       Green for Danger
      </strong>
     </h3>
     <h4 class="transparent">
      <span class="media-identifier media-episode">
       Episode 2
      </span>
     </h4>
     <div class="transparent padding-top-medium">
      <p>
       Directed by Sidney Gilliat • 1946 • United Kingdom
       <br/>
       Starring Trevor Howard, Sally Gray, Alastair Sim
      </p>
      <p>
       In the midst of Nazi air raids, a postman dies on the operating table at a rural English hospital. But was the death accidental? A delightful and wholly unexpected murder mystery, British writer/d...
      </p>
     </div>
    </div>

If the block contains a "•" character it will have both a movie title and a year, otherwise it probably only has a teaser describing the featured movies, and thus is itself not a movie that can be rated. The movie title is extracted from the code block by seeking the line that has a *strong-tag* with no title, style, class attributes:
    
    title = block.find('strong', attrs = {'title':'', 'style':'', 'class':''})

In this example the movie title is *Green for Danger*. 

The year of release is extracted by a split("•") method since the year resides within two "•" symbols, e.g., *" • 1946 • "*. The title and year are then appended to the movie and year lists, and returned by the function. 

Next, the function **selection_choice(movie, target_yr)** operates on each movie and year in the movie and year lists, returning the movie specifications, i.e., the title and year again, along with the movie ID and the rating; e.g. for the *British noir* selection: 

    ('Green for Danger', 1946, '0038577', 7.4)
    ('Odd Man Out', 1947, '0039677', 7.6)
    ('Obsession', 1949, '0041460', 7.3)
    ('The Small Back Room', 1949, '0041886', 7.1)
    ('The Woman in Question', 1950, '0043140', 6.8)
    ('Hell Drivers', 1957, '0051713', 7.2)
    ('Time Without Pity', 1957, '0049856', 6.8)
    ('All Night Long', 1962, '0054614', 7.1)

The **selection_choice()** function works by the IMDdpy package, now known as *Cinemagoer*, which is particulary adapted to the International Movie Database (IMDb) repository of movie data. With the instance *ia = Cinemagoer()*, 
a selection of movies is returned from IMDb by:

    selection = ia.search_movie(movie)

Several movies are returned, since the movie title is generally not unique. The movie ID for each movie in the list is sought, and then with the movie ID, each movie's specifications is examined from the IMDb archives: 

    fetched_movie = ia.get_movie(movieID)

Each fetched movie's release year is compared to the desired one from the Criterion Channel, and if a match is found, it is assumed that the correct movie has been picked, and the rating is extracted, 

    rating = fetched_movie['rating']

and the next movie in the Criterion Channel's list is examined. There are a few error-trapping lines in the **selection_choice()** function since anomalies can occur now and then. The movie title, year of release, movie ID, and rating are then appended to the movies specifications list. 

Recently, movie ratings for movies submitted to the IMDB site would not return, so I created a function, get_rating(movieID) that returns the movie rating using Beautifull Soup. This parses the website information and seeks the property, "og:title," giving the line containing the rating, e.g.:

    <meta content="3 Women (1977) ⭐ 7.7 | Drama, Mystery, Thriller" property="og:title"/>

for the movie, "3 Women." The rating is split out of this line, in this case yielding "7.7."

Next, the movie results are formatted and output to a file, with the function **output_ratings(movies_specs_list, filename)**, *filename* specifies the destination to write to. For the *British noir* example, the output is: 

      Movie (Year)                             Rating
    Foreign Correspondent (1940)                7.4
    The Lady Vanishes (1938)                    7.7
    Young and Innocent (1937)                   6.8
    Sabotage (1936)                             7.0
    The 39 Steps (1935)                         7.6
    The Man Who Knew Too Much (1934)            6.7
    Downhill (1927)                             6.0
    The Lodger: A Story of the London Fog 
     (1927)                                     7.3

The function **chop_string(string, width)** will chop the movie title to fit into the column, as seen by the last entry. 

While the program was inspired by Criterion Channel movies, which requires a yearly - about \\$100, or monthly - about \\$10 fee, a list of movies given the **selection_choice()** function will return the movie specifications, so the functions can be tailored to ones needs. 

These algorithms do not always work: one error is that the movie's year of release may be off by a year, e.g., the Criterion year is *1962*, while the IMDb year is *1961*, which I've seen for some foreign films. Some other times the movie is not listed at the IMDb site; this usually happens again with foreign films, with unmatched movie titles, e.g., when one title may be in English and the other in the host language of the country the movie was made at. I've found this movie rating algorithm to be at least 90% effective with foreign movies, and nearly 100% when rating English language films. 

#### Improvements <a class="anchor" id="improvements"></a>
While this github site is for mostly for cinephile experimentation, I may add some features to help streamline the movie rating processing. 

Since the site works mostly with Criterion Channel movie selections, and most users would probably not be subscribers, I may change the output of movie results from the **criterion_list()** function to be stored as a file, and require the input of movie titles and corresponding release years to **selection_choice()** be read from a file rather than passing the list directly between functions. This would allow a file of movie titles and years composed by another method to be passed to the **selection_choice()** function. 

I would also entertain any thoughts from users to improve the process.
