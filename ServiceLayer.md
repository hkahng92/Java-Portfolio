# View Models, Service layer & TDDs

Example used: rec-coll-jdbctemplate-dao

## View Models

Are object models that are used to present (or take input) data to/from the end user in a form that might be similar or somewhat different than the database structure.

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

**Note:** equals and hashCode methods can be auto-generated using `alt+insert` in windows, then selecting `equals() and hashCode()` from the dropdown menu.
 
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

1. Add Helper Methods to build the ViewModel: Methods that constructs the view model object and returns it

		
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
	
	**This step should be done in parallel with step 5**

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

1. test serviceLayer methods using [TDDs](## TDDs).

## TDDs

### Mocks 