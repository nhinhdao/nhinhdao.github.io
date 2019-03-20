---
layout: post
title:      "Render Events with React Calendar Timeline"
date:       2019-03-19 20:05:09 -0400
permalink:  render_events_with_react_calendar_timeline
---


![Render Events with React Calendar Timeline](https://i.imgur.com/jGUoCHM.png)

If you ever make a **event/project list**, you know that it is best when it is displayed within a **calendar** rather than a plain speadsheet. **Calendar** gives you a clear view of when an event starts and ends. Google calendar or iOs calendar is a good example of that neat display.

With this [Project Management application](https://github.com/nhinhdao/project-management-with-react-redux), I want my users to be able to display their projects the same way.

While there are so many package that helps with this demand, I come upon [React Calendar Timeline](https://github.com/namespace-ee/react-calendar-timeline) and I think its simple display is the best fit for my need.


For your information, [React Calendar Timeline](https://github.com/namespace-ee/react-calendar-timeline) (by **Namespace OÜ**) is a npm package built  to display event lists horizontally in a calendar. Users can zoom, drag, click.. to display or to change dates of any event.

I used the new version called: [New React Calendar Timeline](https://www.npmjs.com/package/new-react-calendar-timeline) but the old version is still working fine with very little differences.

To install, run:

`npm install new-react-calendar-timeline`

It needs [moment](https://momentjs.com/) package to create dates and [interact.js](http://interactjs.io/docs/) package to drag and resize event so please install them as well:

`npm install moment interact.js`

If you want to display events in different colors like me, install [randomcolor](https://www.npmjs.com/package/randomcolor) package to generate random color for each event:

`npm install randomcolor`

Then in your react component, import React Calendar Timeline and its dependencies:

```javascript
import Timeline from 'new-react-calendar-timeline/lib';
import moment from 'moment';
import randomColor from 'randomcolor';
```

Now, you are good to go.

A basic event includes 2 parts: **group** and **item**. **Group** is what you see in the left panel (sometimes on the right hand side or both) and **item** is what you see inside the calendar:

```javascript
const groups = [
  {id: 1, title: 'group 1'},
  {id: 2, title: 'group 2'}
]
 
const items = [
  {id: 1, group: 1, title: 'item 1', start_time: moment(), end_time: moment().add(1, 'hour')},
  {id: 2, group: 2, title: 'item 2', start_time: moment().add(-0.5, 'hour'), end_time: moment().add(0.5, 'hour')},
]
```

![React Calendar Timeline](https://i.imgur.com/q3zwSTR.jpg)

In my project, I fetched the project data from [Rails API back-end](https://github.com/nhinhdao/project-management-railsAPI-backend):

```javascript
class AllProjects extends Component {
  state = { projects: []}

  componentDidMount(){
    const id = localStorage.getItem('userID')
    fetch(`http://localhost:3001/api/v1/allprojects/${id}`)
        .then(response => response.json())
        .then(data => this.setState({projects: data}));
  }

  render(){
    return(
      <React.Fragment>
        <ProjectTimeline projects={this.state.projects} />
        <Route path="/projects/:projectID" render={routerProps => <ProjectPage projects={this.state.projects} {...routerProps} />}/>
      </React.Fragment>
			)
  }
}
```

**React Calendar Timeline** allows you to customise item renderer like drag, zoom, resize, itemClick... Please check out their github repository for more information. I used only the very basic functionality (display only) because I wanted users to go to a different page to edit the events instead:

```javascript
class ProjectTimeline extends Component {
  itemRenderer = ({ item }) => {
    return (
    <Link to={`/projects/${item.id}`}><div onClick={this.handleOpen} style={{backgroundColor: item.bgColor, color: 'black'}}>{item.title}</div></Link>
    )
  }

  render(){
    const groups = [];
    const items = [];

    this.props.projects.forEach(function(project, index){
      groups.push({id: index + 1, title: project.title}); 
      items.push({
        id: project.id, group: index + 1, title: project.description, 
        start_time: moment(project.start_date), end_time: moment(project.end_date),
        canMove: false, canResize: false, canChangeGroup: false,
        bgColor: randomColor({luminosity: "light", format: "rgba", alpha: 0.8})
      })}
    )
    
    return(
      <React.Fragment>
        <div>
          <Button size='small' color='grey'>Projects: {this.props.projects.length}</Button> 
					<Link to={`/newproject`}><Button size='small' color='teal'>Add New Project</Button></Link>
        </div>
        <hr/>
        <Timeline groups={groups}
				items={items}
				sidebarContent={<h3>Project</h3>}
				itemRenderer={this.itemRenderer}
				itemHeightRatio={0.7}
				defaultTimeStart={moment('2019-03-08')}
				defaultTimeEnd={moment('2019-04-08')}
				lineHeight={35}
				/>
      </React.Fragment>
    )
  }
}
```
So after fetching data from [Rails API back-end](https://github.com/nhinhdao/project-management-railsAPI-backend), I looped through the returned data and pushed information into **group** and **item arrays** like above. Each **project** contains *title, discription, start date, end date...* so **group** (left panel) contain id and project titles while **item** (inside calendar) contains descriptions, start date, end date and background color of your choice.

```javascript
    const groups = [];
    const items = [];

    this.props.projects.forEach(function(project, index){
      groups.push({id: index + 1, title: project.title}); 
      items.push({
        id: project.id, group: index + 1, title: project.description, 
        start_time: moment(project.start_date), end_time: moment(project.end_date),
        canMove: false, canResize: false, canChangeGroup: false,
        bgColor: randomColor({luminosity: "light", format: "rgba", alpha: 0.8})
      })}
    )
```

With this setup, you will see a **static calendar** with **project title** in the left and corresponding **project description** in each row. **canMove: false, canResize: false, canChangeGroup: false** will prevent users from dragging and moving projects around, while **bgColor: randomColor({luminosity: "light", format: "rgba", alpha: 0.8})** will give each project a different background color. Keep in mind that the original backgound color is **dark blue** so if you want your items to have light color like that, put this on your CSS file (mine is App.css, then require it in App.js file)

```javascript
div.react-calendar-timeline .rct-items .rct-item{
  background: #ddd;
}
```

Now, if you click on each project, it won't render or display anything. To display each project, I used [Semantic UI Modal](https://react.semantic-ui.com/modules/modal/).

The nice  feature (also annoying sometimes) of react is that if you render child component without using the `exact` keyword, you will also see the parent being rendered. So taking advantage of that, we can render project details in a modal (with blurring background) while the calendar is always being rendered in the background:

Remember itemrenderer function of ProjectTimeline component?

```javascript
  itemRenderer = ({ item }) => {
    return (
    <Link to={`/projects/${item.id}`}>
		<div onClick={this.handleOpen} style={{backgroundColor: item.bgColor, color: 'black'}}>{item.title}</div>
		</Link>
    )
```

When users click on a project inside **Calendar**, they are redirected to **ProjectPage** (`Link to={`/projects/${item.id}`}`).

```javascript
//AllProject component
    <Route path="/projects" component={AllProjects} />
		<Route path="/projects/:projectID" render={routerProps => <ProjectPage projects={this.state.projects} {...routerProps} />}/>
```

```javascript
// ProjectPage component

render(){
    const {project} = this.state
    return(
      <Modal open={this.state.open} dimmer='blurring'>
        <Modal.Header>{project.title}</Modal.Header>
        <Modal.Content image>
          <Modal.Description>
            <Header as="h4">Owner</Header>
            <Label as='a' image><img src={project.owner.image} alt='img'/>{project.owner.username}</Label>
            <Header as="h4">Description</Header>
            <p>{project.description}</p>
            <Form>
              <Form.Group inline>
                <label>Start Date</label>
                <DatePicker onChange={this.handleChange} selected={project.start_date} disabled={true} placeholderText={start_date.toString()}/>
                <label></label>
                <label>End Date</label>
                <DatePicker onChange={this.handleChange} selected={project.end_date} disabled={true} placeholderText={end_date.toString()} />
              </Form.Group>
            </Form>
            <Header as="h4">Tasks</Header>
            <Table basic='very' celled collapsing>
              <Table.Body>
                {project.tasks.map((task, index) => 
                  <Table.Row  key={index}>
                    <Table.Cell>
                      <Header as='h5' image textAlign='center'>
                        <Image src={task.user.image} rounded size='mini' />
                        <Header.Content>{task.user.username}</Header.Content>
                      </Header>
                    </Table.Cell>
                    <Table.Cell>{task.content}</Table.Cell>
                  </Table.Row>
                )}
              </Table.Body>
            </Table>
          </Modal.Description>
        </Modal.Content>
        <Modal.Actions>
          { project.owner.id === parseInt(localStorage.getItem("userID")) &&
          <React.Fragment>
						<Link to={`/editproject/${project.id}`} onClick={this.close}><Button positive>Edit</Button></Link> 
						<Button negative  onClick={this.handleDelete}>Delete</Button>
					</React.Fragment>
					}
          <Link to="/projects"><Button>Close</Button></Link>
        </Modal.Actions>
      </Modal>
    )
  }
```

![Display single event](https://i.imgur.com/s6h9lqR.png)

If you wonder about the `<DatePicker />` inside modal, please read it [here](https://www.npmjs.com/package/react-datepicker). It is a nice way to render date instead of writing it down as text.

In ProjectPage, **edit button** will bring them to **EditPage** `Link to={`/editproject/${project.id}`} `. I intended to not use the same pattern: `Link to={`/projects/${project.id}/edit`}` because if I used `Link to={`/projects/${project.id}/edit`}`, the **edit page** will be rendered underneath **calendar timeline**, which is really ugly. Instead, I want the **edit page** to be rendered alone, not under any parent element.

![Edit Page](https://i.imgur.com/pqBrhJn.png)


However, if users do not want any further action, they can click on **Close button** to go back to the page where they already were: **ProjectTimeline**. Once click, the server will update the url (`to '/projects'`), when the new url no longer matches with the assigned url (`'/projects/:projectID'`), it will be redirected back to **project page** .

If you still feel unclear on how to use **React Calendar Timeline** or any of the packages I have mentioned above, please check out their examples and my github repository for this [Wetask - Project Management application](https://github.com/nhinhdao/project-management-with-react-redux).

I hope you enjoy this touch up. 

Enjoy coding, friends!






