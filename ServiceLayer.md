# View Models, Service layer & TDDs

Example used: rec-coll-jdbctemplate-dao

## View Models

View Models are object models that are used to present (or take input) data to/from the end user in a form that might be similar or somewhat different than the database structure.

* The view model should look exactly like the DTO if we want to present data in a form similar to the database structure, i.e. a class with properties, setters and getters, and with equals and hashcode methods overriden.

* In the view model, we can represent relationships to parent tables in DB using composition. For instance, if the Artist is the parent of the Album table, we can have the AlbumViewModel show the Artist object, instead of just showing the artist_id.

* In the view model, we can also represent relationships to children tables using a list containing the child objects. For instance, we can have the AlbumViewModel show all the Track objects for a given Album inside a list, as shown below.

		private int id;
		private String title;
		private LocalDate releaseDate;
		private BigDecimal listPrice;

		//Parents
		private Artist artist;
		private Label label;
		
		//Children
		private List<Track> tracks;
		
		//Getters and Setters
		.
		.
		.
		
		//equals and hashCode methods
		.
		.
		.

**Note:** equals and hashCode methods can be auto-generated in windows using `alt+insert`, then selecting `equals() and hashCode()` from the dropdown menu.

**DTO Output Example:**

	{
    "id": 1,
    "title": "Album",
    "releaseDate": "2019-12-10",
    "listPrice": 5.52,
    "artistId": "1",
    "labelId": "1"
	}
	
**View Model Output Example:**
	
	{
    "id": 1,
    "title": "Album",
    "releaseDate": "2019-12-10",
    "listPrice": 5.52,
    "artist": {
        "id": 1,
        "name": "Ahmed",
        "instagram": "Insta",
        "twitter": "Twitt"
		},
    "label": {
        "id": 1,
        "name": "labnaem",
        "website": "www"
		},
    "tracks": [
			{
            "id": 1,
            "albumId": 1,
            "title": "track1",
            "runTime": 5
			}
		]
	}
 
## Service Layer

The service layer can validate the view model data, apply other business rules/logic, and translate between the view model and the DTOs associated with the database model. 

### How to code the ServiceLayer class?

1. Add `@Component` annotation to the class.
1. Add DAOs as properties in the class and have a constructer that will instantiate the DAOs

		private AlbumDao albumDao;
		private ArtistDao artistDao;
		private LabelDao labelDao;
		private TrackDao trackDao;

		@Autowired
		public ServiceLayer(AlbumDao albumDao, ArtistDao artistDao, LabelDao labelDao, TrackDao trackDao) {
			this.albumDao = albumDao;
			this.artistDao = artistDao;
			this.labelDao = labelDao;
			this.trackDao = trackDao;
		}

1. Add Helper Methods that will build the ViewModels: methods that constructs the view model object and returns it

		
		private AlbumViewModel buildAlbumViewModel(Album album) {

        // Get the associated artist
        Artist artist = artistDao.getArtist(album.getArtistId());

        // Get the associated label
        Label label = labelDao.getLabel(album.getLabelId());

        // Get the tracks associated with the album
        List<Track> trackList = trackDao.getTracksByAlbum(album.getId());

        // Assemble the AlbumViewModel
        AlbumViewModel avm = new AlbumViewModel();
        avm.setId(album.getId());
        avm.setTitle(album.getTitle());
        avm.setReleaseDate(album.getReleaseDate());
        avm.setListPrice(album.getListPrice());
        avm.setArtist(artist);
        avm.setLabel(label);
        avm.setTracks(trackList);

        // Return the AlbumViewModel
        return avm;

		}
	
1. Implement ServiceLayer Methods: Idealy, one method for each request method in the .yaml file.
	
	**Note: this step should be done in parallel with step 5**
	
	**Important: to saveAlbum, we need to make sure that parents, Labels and Artists in this case, are created first.**

	**Important: to removeAlbum, we need to make sure that all child Tracks are removed first.**

		//
		// Album API
		//
		@Transactional
		public AlbumViewModel saveAlbum(AlbumViewModel viewModel) {

			// Persist Album
			Album a = new Album();
			a.setTitle(viewModel.getTitle());
			a.setReleaseDate(viewModel.getReleaseDate());
			a.setListPrice(viewModel.getListPrice());
			a.setLabelId(viewModel.getLabel().getId());
			a.setArtistId(viewModel.getArtist().getId());
			a = albumDao.addAlbum(a);
			viewModel.setId(a.getId());

			// Add Album Id to Tracks and Persist Tracks
			List<Track> tracks = viewModel.getTracks();

			tracks.stream()
			.forEach(t ->
			{
				t.setAlbumId(viewModel.getId());
				trackDao.addTrack(t);
			});

			tracks = trackDao.getTracksByAlbum(viewModel.getId());
			viewModel.setTracks(tracks);

			return viewModel;
		}

		public AlbumViewModel findAlbum(int id) {

			// Get the album object first
			Album album = albumDao.getAlbum(id);

			return buildAlbumViewModel(album);

		}

		public List<AlbumViewModel> findAllAlbums() {

			List<Album> albumList = albumDao.getAllAlbums();

			List<AlbumViewModel> avmList = new ArrayList<>();

			for (Album album : albumList) {
				AlbumViewModel avm = buildAlbumViewModel(album);
				avmList.add(avm);
			}

			return avmList;
		}

		@Transactional
		public void updateAlbum(AlbumViewModel viewModel) {

			// Update the album information
			Album album = new Album();
			album.setId(viewModel.getId());
			album.setArtistId(viewModel.getArtist().getId());
			album.setLabelId(viewModel.getLabel().getId());
			album.setListPrice(viewModel.getListPrice());
			album.setReleaseDate(viewModel.getReleaseDate());

			albumDao.updateAlbum(album);

			// We don't know if any track have been removed so delete all associated tracks
			// and then associate the tracks in the viewModel with the album
			List<Track> trackList = trackDao.getTracksByAlbum(album.getId());
			trackList.stream()
					.forEach(track -> trackDao.deleteTrack(track.getId()));

			List<Track> tracks = viewModel.getTracks();
			tracks.stream()
					.forEach(t ->
					{
						t.setAlbumId(viewModel.getId());
						t = trackDao.addTrack(t);
					});
		}

		@Transactional
		public void removeAlbum(int id) {

			// Remove all associated tracks first
			List<Track> trackList = trackDao.getTracksByAlbum(id);

			trackList.stream()
					.forEach(track -> trackDao.deleteTrack(track.getId()));

			// Remove album
			albumDao.deleteAlbum(id);

		}

1. test serviceLayer methods using [TDDs](#TDDs).

## TDDs

### My personal approach to designing TDDs

Before we are able to design a test for any method, we need to know three basic information about the method:

1. What does it take as input?

1. What does it return as output?

1. How does the method work?

For example, if we are trying to design a test for saveAlbum method in the service layer, the answers would be:

1. The method takes **AlbumViewModel** as input (with all information about the album except for album_id)

1. It should return **AlbumViewModel** as output (with the same information in addition to the album_id)

1. The service method works by deconstructing the AlbumViewModel into the simpler **AlbumDto** form, and then it uses the **albumDao** to send the **AlbumDto** to the Database to be saved. After being saved in DB, the **albumDao** then returns the **AlbumDto** with album_id. Finally, the service method uses **buildAlbumViewModel** to reconstruct the **AlbumViewModel** and returns it back.

Now, that we have all the information needed to design the test, we approach the actual coding of the test. Keep in mind that our ultimate goal is to compare the expected output of the service to the actual one using `assertEquals` or `assertNull`, depending on the situation.

**Coding the test:**

1. We first hard-code the **input viewModel**.
1. We then pass the input to the service method to return back the **actual output viewModel**
1. We hard-code the **expected output viewModel**
1. We compare the actual with the expected output 

	Final test should look like this:

		@Test
		public void saveAlbum() {
		
		// 1. Hardcoding the input viewModel	
			AlbumViewModel avm = new AlbumViewModel();

			avm.setListPrice(new BigDecimal("14.99"));
			avm.setReleaseDate(LocalDate.of(1999, 05, 15));
			avm.setTitle("Greatest Hits");

			Artist artist = new Artist();
			artist.setInstagram("@RockStar");
			artist.setName("The GOAT");
			artist.setTwitter("@TheRockStar");
			artist = service.saveArtist(artist);

			avm.setArtist(artist);

			Label label = new Label();
			label.setName("Blue Note");
			label.setWebsite("www.bluenote.com");
			label = service.saveLabel(label);

			avm.setLabel(label);

			Track track = new Track();
			track.setRunTime(180);
			track.setTitle("Number 1 Hit!");
			List<Track> tList = new ArrayList<>();
			tList.add(track);

			avm.setTracks(tList);

		//2. Actual Output
			AlbumViewModel fromService = service.saveAlbum(avm); 
			
		//3. Expected Output
			avm.setAlbumId(1) 

			assertEquals(avm, fromService);
		}

1. Separately, we need to [mock](#mocks) the albumDao
1. We also need to instantiate the DAOs and service layer in the @Before Setup() method:
		
		ServiceLayer service;
		AlbumDao albumDao;
		ArtistDao artistDao;
		LabelDao labelDao;
		TrackDao trackDao;

		@Before
		public void setUp() throws Exception {
			setUpAlbumDaoMock();
			setUpArtistDaoMock();
			setUpLabelDaoMock();
			setUpTrackDaoMock();

			service = new ServiceLayer(albumDao, artistDao, labelDao, trackDao);
		}

### Mocks 

Refering back to how the SaveAlbum method works:

1. The service method works by deconstructing the **input AlbumViewModel** into the simpler **AlbumDto** form.
1. It uses the **albumDao** to send the **input AlbumDto** to the Database to be saved.
1. The **albumDao** then returns the **output AlbumDto** with album_id after being saved.
1. Finally, the service method uses **buildAlbumViewModel** to reconstruct the **output AlbumViewModel** and return it back.

We realize that our ServiceLayer test result will have relience on whether the albumDao is working correctly or not. What if we don't have the Dao designed yet?! And what if the serviceLayer method is dependent on an external service that we don't have access to, yet?! And that's why we use mocks, to guarantee that the result of our ServiceLayer tests is solely dependent on the performance of the serviceLayer itself.

#### How to mock the DAO?

1. Declare the mock `albumDao = mock(AlbumDaoJdbcTemplateImpl.class);` 
1. Hard-code the **output DTO**
1. Hard-code the **input DTO**
1. Let the mock know to return **output DTO** when **input DTO** is passed to the **DAO** 

	Final code:

		private void setUpAlbumDaoMock() {
			// 1. 
			albumDao = mock(AlbumDaoJdbcTemplateImpl.class);
			
			// 2. Hard-coded output DTO
			Album album = new Album();
			album.setId(1);
			album.setArtistId(45);
			album.setLabelId(10);
			album.setTitle("Greatest Hits");
			album.setListPrice(new BigDecimal("14.99"));
			album.setReleaseDate(LocalDate.of(1999, 05, 15));

			// 3. Hard-coded input DTO
			Album album2 = new Album();
			album2.setArtistId(45);
			album2.setLabelId(10);
			album2.setTitle("Greatest Hits");
			album2.setListPrice(new BigDecimal("14.99"));
			album2.setReleaseDate(LocalDate.of(1999, 05, 15));

			// 4. Return output DTO when input is passed through albumDao
			doReturn(album).when(albumDao).addAlbum(album2);
		}
		
**Note:** For the mock to work, we need to make sure that information used to hard-code **input** and **output DTOs** are exactly the same used for hard-coding **input ViewModel** and **Output ViewModel**, i.e. the same album_id, artist_id, etc. 

*The same approch could be used to design DAO TDDs, except that we wouldn't need to worry about having mocks.*

#### How to mock an external microservice?

* Case (1): An external service that is called using RestTemplate and DiscoveryClient

	First, the setUp:
	
		private TaskerServiceLayer service;
		private TaskerDao taskerDao;
		private RestTemplate restTemplate;
		private DiscoveryClient discoveryClient;

		@Value("${adServerServiceName}")
		private String adServerServiceName;

		@Value("${serviceProtocol}")
		private String serviceProtocol;

		@Value("${servicePath}")
		private String servicePath;

		@Before
		public void setUp() throws Exception {
			
			// We call the mock methods here
			setUpTaskerDaoMock();
			setUpRestTemplateMock();
			setUpDiscoveryClientMock();

			// We use a custome made service layer constructor to pass on the mocked classes and properties
			service = new TaskerServiceLayer(taskerDao, discoveryClient, restTemplate, adServerServiceName, serviceProtocol, servicePath);

		}
		
	Second, we mock the RestTemplate:
	
		private void setUpRestTemplateMock() {
		
			restTemplate = mock(RestTemplate.class);

			doReturn("BOGO large 2 topping pizzas!").when(restTemplate).getForObject("http://localhost:6107/ad", String.class);
		}

	Finally, we mock the discoveryClient:
	
		private void setUpDiscoveryClientMock() {
			
			discoveryClient = mock(DiscoveryClient.class);
			
			// discoveryClient returns a LinkedList of DefaultServiceInstances with hostName and portNumber
			List<ServiceInstance> instances = new LinkedList<>();

			DefaultServiceInstance defaultServiceInstance = new DefaultServiceInstance("","","localhost",6107,true);

			instances.add(defaultServiceInstance);

			doReturn(instances).when(discoveryClient).getInstances("adserver-service");
		}

* Case (2): An external service that is called using Feign Client

		Still figuring this one out!

### Using ServiceLayer in the controller

To use the ServiceLayer in the controller, we simply `@Autowired` the serviceLayer in the controller class to use its methods:

	@Autowired
    ServiceLayer serviceLayer;

    @PostMapping
    @ResponseStatus(value = HttpStatus.ACCEPTED)
    public AlbumViewModel createAlbum(@RequestBody @Valid AlbumViewModel albumViewModel){
        return serviceLayer.saveAlbum(albumViewModel);
    }

    @GetMapping("/{id}")
    @ResponseStatus(HttpStatus.OK)
    public AlbumViewModel getAlbum(@PathVariable("id") int albumId){
        return serviceLayer.findAlbum(albumId);
    }

#### [Go Back](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)