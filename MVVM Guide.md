**MVVM Guide:**

**Retrofit Cilent**

    object RetrofitClient {
    val instance : WebService by lazy {
        val retrofit = Retrofit.Builder()
            .baseUrl(Constants.BASE_DOMAIN_2)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
        return@lazy retrofit.create(WebService::class.java)
    }
    }
    
   
  
  
   **Web Service**
    
    interface WebService {

    @GET("Your URL")
    suspend fun getPerks(): Perks 
    }
    
   **View Model** 
  
 ``` class PerksViewModel : ViewModel() {
    
    private var _perksList = MutableLiveData<Perks>()
    val perksList: LiveData<Perks>
        get() = _perksList

    init {
        fetchPerks()
    }

    private fun fetchPerks() {
        viewModelScope.launch {
            try {
                val data = RetrofitClient.instance.getPerks()
                _perksList.value = data

            } catch (e: Exception) {
                Timber.e(e)
            }
        }
    }
}
```

**Fragment / Activity Class**
    
    class SelectFragment : Fragment(R.layout.fragment_select) {

    private lateinit var progress: ProgressBar
    private lateinit var viewModel: PerksViewModel
    private var fakeList: ArrayList<PerksDatum>? = arrayListOf()
    private lateinit var adapter: PerksAdapter
    private lateinit var binding: FragmentSelectBinding

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = FragmentSelectBinding.inflate(layoutInflater)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel = ViewModelProvider(this).get(PerksViewModel::class.java)
        init(view)
    }

    private fun init(view: View) {

        binding.fragmentSelectRecyclerView.layoutManager = LinearLayoutManager(activity)

        viewModel.perksList.observe(viewLifecycleOwner, Observer {
            adapter.addList(it.data as ArrayList<PerksDatum>)
        })

        adapter = PerksAdapter {
            navigateFragment(
                view,
                it.image,
                it.buttonText,
                it.title,
                it.descriptionWithoutHtml,
                it.link
            )
        }
        binding.fragmentSelectRecyclerView.adapter = adapter
    }

    private fun navigateFragment(
        view: View,
        imageView: String?,
        buttonText: String?,
        title: String?,
        description: String?,
        link: String?
    ) {

        val bundle = bundleOf(
            "ImageView" to imageView, "ButtonText" to buttonText,
            "Title" to title, "Description" to description, "Link" to link
        )

        Navigation.findNavController(view)
            .navigate(R.id.action_nav_select_to_selectEventDescriptionFragment, bundle)
    }

}

**Adapter Class**

```class PerksAdapter(private val onClick: (objDatum: PerksDatum) -> Unit) :
    ListAdapter<PerksDatum, PerksAdapter.PerksViewholder>(
        PerksDiffUtilCallback()
    ) {

override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PerksViewholder {
        val layoutInflater = LayoutInflater.from(parent.context)
            .inflate(R.layout.select_design, parent, false)
        return PerksViewholder(layoutInflater)
    }

    override fun onBindViewHolder(holder: PerksViewholder, position: Int) {
        holder.bind(position)
        holder.itemView.setOnClickListener {
            onClick(getItem(position))
        }
    }

    inner class PerksViewholder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val perksImage: ImageView = itemView.findViewById(R.id.perk_image)
        private val perksTitle: TextView = itemView.findViewById(R.id.perks_title)
        private val perksDescription: TextView = itemView.findViewById(R.id.perks_description)

        fun bind(position: Int) {
            val list = getItem(position)
            perksTitle.text = list.title
            perksDescription.text = list.excerpt
            Glide.with(itemView.context).load(list.image).into(perksImage)
           // perksImage.load(list.image)
        }
    }

    class PerksDiffUtilCallback : DiffUtil.ItemCallback<PerksDatum>() {
        override fun areItemsTheSame(oldItem: PerksDatum, newItem: PerksDatum): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: PerksDatum, newItem: PerksDatum): Boolean {
            return oldItem == newItem
        }

    }
} 
```


