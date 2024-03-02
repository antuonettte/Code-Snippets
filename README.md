# Code-Snippets
Snippets of code I've created


The below snippet is from a project I was working on where I had to create endpoints for doing CRUD operations on data after validating the user.

```Java
    @RequestMapping("/{ideaId}")
    public ResponseEntity<?> getIdeaById(
            @PathVariable(value="ideaId") Integer ideaId,
            @CurrentUser User currentUser) {

        CurrentAppUser currentAppUser = currentAppUserService.getCurrentAppUser();

        logger.info("Fetching Idea [{}]; User: \"{}\";", ideaId, currentAppUser.getUserForLogging());

        Idea idea = ideaService.findById(ideaId);

        if (idea == null) {
            logger.error("Idea [{}] not found. User: \"{}\";", ideaId, currentAppUser.getUserForLogging());
            return new ResponseEntity<>(
                    new ErrorDetails("IdeaExtended with id [" + ideaId + "] not found."),
                    HttpStatus.NOT_FOUND
            );
        }
        if (currentAppUser.isAdmin()
                || currentAppUser.isReviewer()
                || currentAppUser.getUser().getAppUserId().equals(idea.getOwnerId())) {
            return new ResponseEntity<>(idea, HttpStatus.OK);
        }
        else {
            logger.error(
                    "Only admins and the original idea submitter can load this idea. " +
                            "IdeaId: [{}]; OwnerId: [{}]; User: \"{}\"",
                    ideaId,
                    currentAppUser.getUser().getAppUserId()
            );

            return new ResponseEntity<>(
                    new ErrorDetails("Only admins and the idea submitter can load this idea."),
                    HttpStatus.FORBIDDEN
            );
        }
    }

    @PostMapping()
    public ResponseEntity<Idea> createIdea(
            @RequestBody Idea idea,
            @CurrentUser User currentUser) {

        CurrentAppUser currentAppUser = currentAppUserService.getCurrentAppUser();

        logger.info("Creating idea. User: \"{}\";", currentAppUser.getUserForLogging());

        Idea ideaSaved = ideaService.save(idea);
        return new ResponseEntity<>(ideaSaved, HttpStatus.CREATED);
    }

    @PutMapping("/{ideaId}")
    public ResponseEntity<?> saveIdea(
        @PathVariable(value="ideaId") Integer ideaId,
        @RequestBody Idea idea,
        @CurrentUser User currentUser)
    {
        CurrentAppUser currentAppUser = currentAppUserService.getCurrentAppUser();

        logger.info("Fetching & Updating Idea [{}]. User: \"{}\";", ideaId, currentAppUser.getUserForLogging());

        Idea currentIdea = ideaService.findById(ideaId);

        if (currentIdea == null) {
            logger.error(
                    "Unable to update. Idea with id [{}] not found. User: \"{}\";",
                    ideaId,
                    currentAppUser.getUserForLogging()
            );

            return new ResponseEntity<>(
                new ErrorDetails("Unable to update. Idea [" + ideaId + "] not found."),
                HttpStatus.NOT_FOUND
            );
        }

        if (idea.getIdeaId() != null && !idea.getIdeaId().equals(ideaId)) {
            logger.error(
                    "IdeaId: [{}] in the api route does not match the IdeaId " +
                            "in the request body: [{}]. User: \"{}\";",
                    ideaId,
                    idea.getIdeaId(),
                    currentAppUser.getUserForLogging()
            );

            return new ResponseEntity<>(
                    new ErrorDetails(
                            "IdeaId: [" + ideaId + "] in the api route does not " +
                                    "match the IdeaId in the request body: [" + idea.getIdeaId() + "]"
                    ),
                    HttpStatus.BAD_REQUEST
            );
        }

        if (currentAppUser.isAdmin()
            || currentAppUser.isReviewer()
            || currentAppUser.getUser().getAppUserId().equals(currentIdea.getOwnerId())) {

            // set primary key if it was left off the model in the request body
            if (idea.getIdeaId() == null) {
                idea.setIdeaId(ideaId);
            }

            Idea savedIdeaExtended = ideaService.save(idea);
            return new ResponseEntity<>(savedIdeaExtended, HttpStatus.OK);
        }
        else {
            logger.error(
                    "Only admins and the idea submitter can update an idea. " +
                            "IdeaId: [{}]; OwnerId: [{}]; User: \"{}\";",
                    ideaId,
                    currentIdea.getOwnerId(),
                    currentAppUser.getUserForLogging()
            );

            return new ResponseEntity<>(
                    new ErrorDetails("Only admins and the idea submitter can update an idea."),
                    HttpStatus.FORBIDDEN
            );
        }
    }

```


This next snippet fetches data from a webpage, parses the HTML to find a specific data table, extracts the table's rows into a list of lists (each sub-list represents a row), and then writes this data to a CSV file.

```python 
import requests
from bs4 import BeautifulSoup
import csv

def scrape_website(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    return soup

def parse_data(soup):
    data = []
    table = soup.find('table', attrs={'class':'data-table'})
    table_rows = table.find_all('tr')

    for tr in table_rows:
        td = tr.find_all('td')
        row = [tr.text.strip() for tr in td if tr.text.strip()]
        if row:
            data.append(row)
            
    return data

def save_to_csv(data, filename):
    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerows(data)

def main():
    url = 'http://example.com' 
    soup = scrape_website(url)
    data = parse_data(soup)
    save_to_csv(data, 'output.csv')

if __name__ == '__main__':
    main()

```



This snippet uses React hooks and retrieves data from an API, manages it with local state, and provides a way to filter that data. It also uses context to get a theme value.

```Javascript
import React, { useState, useEffect, useContext } from 'react';
import axios from 'axios';
import { ThemeContext } from './ThemeContext';

const API_ENDPOINT = 'https://jsonplaceholder.typicode.com/posts';

const PostList = () => {
  const [posts, setPosts] = useState([]);
  const [filter, setFilter] = useState('');
  const theme = useContext(ThemeContext);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        const response = await axios.get(API_ENDPOINT);
        setPosts(response.data);
      } catch (error) {
        console.error('Failed to fetch posts:', error);
      }
    };

    fetchPosts();
  }, []);

  const handleFilterChange = event => {
    setFilter(event.target.value);
  };

  const filteredPosts = posts.filter(post =>
    post.title.toLowerCase().includes(filter.toLowerCase())
  );

  return (
    <div style={{ backgroundColor: theme.background, color: theme.foreground }}>
      <input type="text" value={filter} onChange={handleFilterChange} placeholder="Filter posts" />
      <ul>
        {filteredPosts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default PostList;

```
