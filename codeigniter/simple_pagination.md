# Getting started with pagination in CodeIgniter

## Intro

Sometimes your database can return a lot of data. Like hundreds (or even thousands) of rows that may need to be outputted to the user. In these kinds of situations, it's good to use pagination, which codeigniter has a built-in library for.

Pagination is the process of dividing this data into chunks, like pages in a book. On your webpage, this often looks like a series of clickable number links, but can also be formatted to just previous and next links. Behind the scenes though, pagination works with SQL's [LIMIT and OFFSET clauses](http://www.w3schools.com/php/php_mysql_select_limit.asp). These two clauses, in essence, specify a range of results (database rows) to output.

In CodeIgniter,  the [pagination library](https://www.codeigniter.com/user_guide/libraries/pagination.html) takes care of figuring out how many pagination links are required, and inserts the OFFSET clause value as a method argument to the controller method you are calling it from. Here is an example of just one pagination link

```html 
<a href="/language/read/20" data-ci-pagination-page="2">2</a> 
```

## How to do it

Here is how I attempted to use pagination.

**Controller**

```php
public function read($pagination_offset = 0)
{
  $this->load->library('pagination');
  $pagination_config['base_url'] = base_url('language/read'); // the controller you are calling
  $pagination_config['per_page'] = 20; // sql query LIMIT number

  $this->data['languages'] = $this->language_model->get_languages($pagination_config['per_page'], $pagination_offset); // your database call with LIMIT and OFFSET arguments
  $total_query_rows = $this->db->query('SELECT id FROM languages')->num_rows(); // what the database would have yielded without the LIMIT and OFFSET clauses (*see Notes)

  $pagination_config['total_rows'] = $total_query_rows;
  $this->pagination->initialize($pagination_config); // prepares the pagination based on your configurations

  $this->data['pagination'] = $this->pagination->create_links(); // outputs the pagination as a string (*see Notes)
  $this->load->view('language', $data);
}
```

**Model**

```php
public function get_languages($limit, $pagination_offset)
{
  $query = $this->db->query("
  SELECT
      languages.id AS id,
      languages.title AS language,
      country_code,
      COUNT(courses.id) AS number_of_courses
    FROM languages
    LEFT JOIN courses
    ON languages.id = courses.language_id
    GROUP BY languages.title
    LIMIT $limit OFFSET $pagination_offset
    ");
    return $query->result();
  }
```

## Notes

The three most important things here are

* `$pagination_config['base_url']` the controller method to call.
* `$pagination_config['per_page']` the SQL LIMIT or how many results per page you want to see.
* `$pagination_config['total_query_rows']` all the rows in the results set, so the pagination library can figure out how many links to create.

In `$total_query_rows` instead of writing the whole query without LIMIT and OFFSET, I wrote a much simpler one because I knew it would return the same number of rows every time.

If there is no pagination to show, create_links() will output an empty string.

In the model, you can of course use the query class library to pass LIMIT and OFFSET clauses.

## Changing the view output

Above is the minimum you need to get pagination working, but it's likely you will want to style or customise your output. To do this you can edit the default preferences which are passed to the `initialize()` function. I prefer to do this by creating a `pagination.php` file in the `config` folder. Here is an example styled with the [W3.CSS](http://www.w3schools.com/w3css/) front-end CSS framework

**pagination.php**

```php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

$config['full_tag_open'] = '<ul class="w3-pagination w3-border w3-light-grey">';
$config['full_tag_close'] = '</ul>';

$config['first_link'] = 'First'; // might require more than 5 pages by default to show up
$config['first_tag_open'] = '<li>';
$config['first_tag_close'] = '</li>';
// $config['first_url'] = ''; // An alternative URL to use for the “first page” link.

$config['last_link'] = 'Last';
$config['last_tag_open'] = '<li>';
$config['last_tag_close'] = '</li>';

$config['next_link'] = '&raquo;';
$config['next_tag_open'] = '<li>';
$config['next_tag_close'] = '</li>';

$config['prev_link'] = '&laquo;';
$config['prev_tag_open'] = '<li>';
$config['prev_tag_close'] = '</li>';

$config['cur_tag_open'] = '<li><b><a class="w3-green">';
$config['cur_tag_close'] = '</a></b></li>';

$config['num_tag_open'] = '<li>';
$config['num_tag_close'] = '</li>';
```

**View**

Finally, you'll need to ouput it somewhere in your view. 

```php
<?php if (!empty($pagination)): ?>
  <div class="w3-container">
    <div class="w3-card-2 w3-padding w3-light-grey">
      <?php echo $pagination; ?>
    </div>
  </div>
<?php endif; ?>
```
