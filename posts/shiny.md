# shiny

~~~{r}
library(shiny)

# Define UI for application that draws a histogram
bike <- read_csv('bikeshare.csv')
bike_add_time <- bike %>% mutate(hour = format(datetime, "%H"))
datetime_info <- bike_add_time %>% distinct(hour) %>% arrange(hour) %>% select(hour)

ui <- fluidPage(
   
   # Application title
   titlePanel("Old Faithful Geyser Data"),
   
   # Sidebar with a slider input for number of bins
   sidebarLayout(
      sidebarPanel(
          selectInput("dataSelect"," :", choices=datetime_info)
      ),
     
      # Show a plot of the generated distribution
      mainPanel(
         plotOutput("distPlot")
      )
   )
)

# Define server logic required to draw a histogram
server <- function(input, output) {
   
   output$distPlot <- renderPlot({
     bike <- bike_add_time %>% filter(hour == input$dataSelect)
     
     ggplot(filter(bike, workingday==1), aes(x=hour, y=count)) +
     geom_point()
   })
}

# Run the application
shinyApp(ui = ui, server = server)
~~~

