package main import ( 
    "encoding/json"     "fmt" 
    "html/template" 
    "log" 
    "net/http" 
    "net/url"     "os" 
    "strconv" 
    "time" ) var tpl = template.Must(template.ParseFiles("index.html")) var apiKey = "5db4d29d8e334655862b2a42bcb17307" // Results final struct for the response from the API type Results struct { 
    Status       string    `json:"status"` 
    TotalResults int       `json:"totalResults"` 
    Articles     []Article `json:"articles"` 
} 
type Article struct { 
    Source struct { 
        ID   interface{} `json:"id"` 
        Name string      `json:"name"` 
    } `json:"source"` 
    Author      string    `json:"author"` 
    Title       string    `json:"title"` 
    Description string    `json:"description"` 
    URL         string    `json:"url"` 
    URLToImage  string    `json:"urlToImage"` 
    PublishedAt time.Time `json:"publishedAt"` 
    Content     string    `json:"content"` 
} 
// FormatPublishedDate formats date to normal form func (a *Article) FormatPublishedDate() string {     year, month, day := a.PublishedAt.Date()     return fmt.Sprintf("%v %d, %d", month, day, year) 
} 
// Search query type Search struct { 
    SearchKey  string 
    NextPage   int 
    TotalPages int 
Results    Results 
} func (s *Search) GoToNext() int {     return s.NextPage + 1 
} func (s *Search) PreviousPage() int {     return s.NextPage - 1 
} func indexHandler(w http.ResponseWriter, req *http.Request) {     tpl.Execute(w, nil) 
} func searchHandler(w http.ResponseWriter, r *http.Request) {     u, err := url.Parse(r.URL.String())     if err != nil { 
        w.WriteHeader(http.StatusInternalServerError) 
        w.Write([]byte("Internal server error"))         return     }     params := u.Query()     searchKey := params.Get("q")     page := params.Get("page")     if page == "" {         page = "1" 
}     search := &Search{}     search.SearchKey = searchKey 
    //converts page variable to integer and stores it in next     next, err := strconv.Atoi(page)     if err != nil {         http.Error(w, "Unexpected server error", http.StatusInternalServerError)         return     }     if searchKey == "" {         err = tpl.Execute(w, search)         if err != nil {             log.Println(err)             return 
        }     }     search.NextPage = next     pageSize := 20 
    //Sprintf is used to format stirng with given variables 
    endpoint 	:= 
fmt.Sprintf("https://newsapi.org/v2/everything?q=%s&pageSize=%d&page=%d&apiKey=%s& sortBy=publishedAt&language=en",         url.QueryEscape(search.SearchKey), pageSize, search.NextPage, apiKey) resp, err := http.Get(endpoint)     if err != nil { 
        w.WriteHeader(http.StatusInternalServerError)         return     }     defer resp.Body.Close()     if resp.StatusCode != 200 { 
        w.WriteHeader(http.StatusInternalServerError)         return     }     err = json.NewDecoder(resp.Body).Decode(&search.Results)     if err != nil { 
        w.WriteHeader(http.StatusInternalServerError)         return     }     search.TotalPages = search.Results.TotalResults / pageSize     if search.Results.TotalResults%pageSize != 0 {         search.TotalPages++ 
    }     err = tpl.Execute(w, search)     if err != nil {         log.Println(err) 
    } 
} func main() {     mux := http.NewServeMux()     assets := http.FileServer(http.Dir("css"))     mux.Handle("/css/", http.StripPrefix("/css/", assets))     mux.HandleFunc("/search", searchHandler)     mux.HandleFunc("/", indexHandler)     port := os.Getenv("PORT")     http.ListenAndServe(":"+port, mux) 
} 
<!DOCTYPE html> 
<html lang="en" dir="ltr">   <head> 
    <meta charset="utf-8"> 
    <title></title> 
    <link rel="stylesheet" href="style.css"> 
    <meta name="viewport" content="width=device-width, initial-sclae=1.0"> 
    <style>         body{   margin: 0;    padding: 0;    font-family: "open sans",sans-serif;  background: #F4F4F4; 
  background-size: cover;   height: 100%; 
 
} .about-section{   width: 70%; background:#fff;   padding: 40px 0;   margin-left: auto;   margin-right:auto;   margin-top: 170px; box-shadow: rgba(0, 0, 0, 0.15) 0px 5px 15px 0px; border-radius: 30px; border-left: 50px solid #3CEDA7; border-right: 50px solid #3CEDA7; 
} .inner-width{   max-width: 1000px;   overflow: hidden;   padding: 0 20px;   margin: auto; } 
.about-section h1{ 
  text-align: center; } .border{   width: 100px;   height: 3px;   background: #3CEDA7;   margin: 30px auto; } .about-section-row{   display: flex;   flex-wrap: wrap; } .about-section-col{   flex: 50%; } .about{   padding-right: 30px; } .about p{   text-align: justify;   margin-bottom: 20px;   color: #7E7C7A;   font-size: 17px; 
} .about a{   display: inline-block;   color: #7E7C7A;   text-decoration: none;   border: 2px solid #3CEDA7;   border-radius: 24px;   padding: 8px 40px;   transition: 0.4s linear; } .about a:hover{   color: #fff;   background: #3CEDA7; 
} .skills{   padding-left: 30px; 
} .skill{   margin-bottom: 10px; 
} .title{   color: #7E7C7A 
} 
.progress{   width: 100%;   height: 12px;   background: #ddd;   border-radius: 12px; } .progress-bar{   height: 12px;   background: #3CEDA7;   border-radius: 12px; 
} .p1{   width: 90%; } .p2{   width: 70%; } .p3{   width: 50%; } .progress-bar span{   float: right;   margin-right: 6px;   line-height: 13px;   color: #fff;   font-size: 12px; 
} 
@media screen and (max-width:700px) {   .about-section-col{     flex: 100%;     margin: 10px 0;   }   .about,.skills{     padding: 0;   }   .about{     text-align: center; 
  } 
} 
    </style> 
  </head> 
  <body> 
      <div class="about-section"> 
        <div class="inner-width"> 
          <h1>About Project</h1> 
          <div class="border"></div>           <div class="about-section-row"> 
            <div class="about-section-col"> 
              <div class="about"> 
                <p> 
                    Artificial Intelligence (AI) is changing all aspects of communications and journalism as automatic processes are being introduced into all facets of classical journalism: investigation, content production, and distribution. Traditional human roles in these fields are being replaced by automatic processes and robots. 
                    The first section of this book focuses on a discussion of AI, the new emerging field of robot journalism, and the opportunities that AI limitations create for human journalists. The second section offers examples of the new journalism storytelling that empower human journalists using new technologies, new applications, and AI tools. While this book focuses on journalism, the discussion and conclusions are relevant to all content creators, including professionals in the advertising industry, which is a major main source of support for journalism. 
                </p> 
                <!-- <a href="#">Read More</a> --> 
              </div> 
            </div> 
            <div class="about-section-col"> 
              <div class="skills"> 
                <div class="skill"> 
                 <img src="assets/images/jor.jpeg" alt=""> 
                </div> 
              </div> 
            </div> 
          </div> 
        </div> 
      </div> 
  </body> 
</html> 
 
<!DOCTYPE html> 
<html lang="en"> 
    <head> 
        <meta charset="UTF-8"> 
        <!-- Primary Meta Tags --> 
        <title>News-Search_App</title> 
        <meta name="description" content="Search any topic and get latest news">  
        <!-- Open Graph / Facebook --> 
        <meta property="og:type" content="website"> 
        <meta property="og:url" content="https://news--fetch.herokuapp.com/"> 
        <meta property="og:title" content="News-Search_App by Factorial"> 
        <meta property="og:description" content="Search any topic and get latest news"> 
        <meta property="og:image" content="https://metatags.io/assets/meta-tags-
16a33a6a8531e519cc0936fbba0ad904e52d35f34a46c97a2c9f6f7dd7d336f2.png">  
        <!-- Twitter --> 
        <meta property="twitter:card" content="summary_large_image"> 
        <meta property="twitter:url" content="https://news--fetch.herokuapp.com/"> 
        <meta property="twitter:title" content="News-Search_App by Factorial"> 
        <meta property="twitter:description" content="Search any topic and get latest news"> 
        <meta property="twitter:image" content="https://metatags.io/assets/metatags-16a33a6a8531e519cc0936fbba0ad904e52d35f34a46c97a2c9f6f7dd7d336f2.png">  
        <link 	rel="stylesheet" 
href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"> 
    </head> 
    <body> 
        <div class="container" style="padding-top: 120px; width: 50%;"> 
            <div class="jumbotron"> 
                <div class="row"> 
                    <div class="col-4 d-flex justify-content-center text-center"> 
                        <p 	style="font-weight:bolder;text-transform: uppercase;">design and implementation of an artificial intelligencent robotic journalism system</p> 
                        <img src="assets/images/logo.jpeg" style="height:200px;" alt=""> 
                        <br> 
                         
                        <h3 class="text-uppercase">adeyemi ibukunoluwa S</h3> 
                        <h3 class="text-uppercase">26013819</h3> 
                    </div> 
                </div> 
            </div> 
               
          </div>         </main> 
        <script> 
            // var delayInMilliseconds = 5000; //1 second             // setTimeout(function() { 
            //      window.location.replace("main.html"); 
            // }, delayInMilliseconds); 
            
        </script> 
    </body> 
</html> 

