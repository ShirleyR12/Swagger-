# Swagger-
WEB API - ASP.NET 5.0 

------
Aula do meu curso WEB API 
Passo a passo

- SWAGGER É UMA INTERFACE QUE PERMITE TESTAR A APLICAÇÃO SEM UTILIZAR UM CLIENT EXTERNO, ALÉ DISSO É UMA FORMA DE DOCUMENTAR SUA API 

1° -cria o projeto 
2° deleta o w.. do controle e da raiz do projeto
3° clica com o direito no noe do projeto > nugets e busque e baixe OS PACOTES p/ utilizar o context e entiteframew.. etc

pelo package Manager Console digite: Intall-Package Microsoft.EntityFrameworkCore ouu

 'MicrosoftEntityFrameworkCore'
  EntityFrameworkCore.Tools   --> esse pacote nos permite executar as migrations
  EntityFrameworkCore.SqlServer

4°  add Model na raiz do projeto 
5° add uma class Book
6° crie os campos
 int id 
 string Title 
 string Author 
 string Description
 7° Crie a classe (BookContext) na Model pra mapeamento das entidades com o banco
 8° Herde a : DbContext
  - crie:
   public BookContext(DbContextOptions<BookContext> options): base(options)
   {
	DataBase.EnsureCreated();
   }
   
   public DbSet<Book> Books {get; set;}

9° Vai na classe STARTUP
 - Insere em ConfigureServices: services.AddDbContext<BookContext>(x => x.UseSqlite("Data source=books.db"));
 
10° Crie uma nova pasta raiz com o nome : Repositories
11° Crie uma interface nela com o nome: IBookRepository
12° Dentro dela crie as assinaturas:
	 Task<IEnumerable<Book>> Get();
	 Task<Book> Get(int id);
	 Task<Book> Create(Book book);
	 Task Update (Book book);
	 Task Delete(int id);
	 
13° Crie uma nova classe na 'Repositories' com o nome: 'BookRepository'
14° herde a interface  'BookRepository :IBookRepository'
15° Importe a context e crie o contrutor
	private readonly BookContext _context;
	
	public BookRepository(BookContext context){
		_context = context;
		}
16° Implementa os métodos

public async Task<Book> Create(Book book)
{
	_context.Book.Add(book);
	await _context.SaveChangesAsync();
	
	return book;
}

public async Task Delete(int id)
{
	var bookToDelete = await _context.Book.FindAsync(id);
	_context.Book.Remove(bookToDelete);
	await _context.SaveChangesAsync();
	
}
	 
public async Task<IEnumerable<Book>> Get()
{
	return await _context.Book.ToListAsync();
	
}

public async Task<Book> Get(int id)
{
	return await _context.Book.FindAsync();
	
}
public async Task Update(Book book)
{
	_context.Entry(book).State = EntityState.Modified;
	await _context.SaveChangesAsync();
	
}
  

17° No Startup  dentro do configureServices inseir: services.AddScoped<IBookRepository, BookRepository>();
18° NO controller cirar a class 'BooksController'
19° Importar a interface |  
 private readonly IBookRepository  _bookRepository;
 
20° Criar construtor

public BooksController(IBookRepository bookRepository)
{
	_bookRepository = bookRepository;
}

21° criar endpoint

[HttpGet]
public async Task<IEnumerable<Book>> GetBooks()
{
	return await _bookRepository.Get();
}

[HttpGet("{id}")]
public async Task<ActionResult<Book>> GetBooks(int id)
{
	return await _bookRepository.Get(id);
}

[HttpPost]
public async Task<ActionResult<Book>> PostBooks([FromBdy] Book book)
{
	var newBook = await _bookRepository.Create(book);
	return CreateAtAction(nameof(GetBooks), new {id = newBook.Id } , newBook);
}

[HttpPut]
public async Task<ActionResult> PutBooks(int id, [FromBdy] Book book)
{
	if(id != book.id)
	{
		return BadRequest();
	}
	
	await _bookRepository.Update(book);
	return NoContext();
}

[HttpDelete("{id}")]
public async Task<ActionResult> Delete(int id)
{
	var bookToDelete = await _bookRepository.Get(id);
	if (bookToDelete == null)
		return NoFound();
		
	await _bookRepository.Delete(bookToDelete.Id);
	return NoContext();
}

------------------------------------------------------------------
*comandos para migration:
Add-Migration Book
