<?php

namespace App\Http\Controllers;
use DB;
use Carbon\Carbon;
use App\Models\User;
use App\Models\Comment;
use App\Traits\Commentable;
use Illuminate\Http\Request;
use App\Traits\ValidatesRouteParams;
use Illuminate\Database\Eloquent\Model;
use App\Transformers\CommentTransformer;
use App\Traits\AcceptsRequestRelationships;

class CommentController extends Controller
{
    use ValidatesRouteParams, AcceptsRequestRelationships;

    public function __construct()
    {
        $this->authorizeResource(Comment::class);
    }

    /**
     * @param Comment $comment
     *
     * @return array
     */
    public function view(Comment $comment)
    {
        return $this->prepare($comment);
    }

    /**
     * @param Request           $request
     * @param Model|Commentable $model
     *
     * @return array
     */
    public function create(Request $request, Model $model)
    {
        $this->validateRouteParams(['model' => 'uses:'.Commentable::class]);
        $this->validate($request, ['content' => 'required|min:1']);

        $check = Comment::where('commentable_id', '=', $request->get('post'))->where('enabled', '=', 0)->where('draft', '=', 1)->where('user_id', '=', auth()->user()->id)->get();

        if($check->count() > 0)
        {
          $deleteRecord = Comment::where('commentable_id', '=', $request->get('post'))->where('enabled', '=', 0)->where('draft', '=', 1)->where('user_id', '=', auth()->user()->id)->delete();

        }
        //else if($check->count() <= 0)
        $comment = $model->attachComment(auth()->user(), $request->get('content'), $request->get('enabled'), $request->get('draft'));

        return $this->prepare($comment);
    }

    public function stats(User $user)
    {
        $includes = 'views,votes,comments';
        $excludes = 'content,tags';
        $range    = request('filter.range');
        $comment    = $user->publishedcomment()->latestFirst();

        if ($range) {
            $end_date = Carbon::now();

            switch ($range) {
                case 'LAST_THREE_MONTHS':
                    $start_date = Carbon::now()->subMonths(3);
                    break;

                case 'LAST_THIRTY_DAYS':
                    $start_date = Carbon::now()->subDays(30);
                    break;

                case 'LAST_SEVEN_DAYS':
                    $start_date = Carbon::now()->subDays(7);
                    break;
            }
        }

        if (isset($start_date) && isset($end_date)) {
            $comment->whereBetween('created_at', [$start_date, $end_date]);
        }

        return fractal()
            ->collection($comment->get())
            ->transformWith(new CommentTransformer)
            ->parseIncludes($includes)
            ->parseExcludes($excludes)
            ->toArray();
    }

    public function favorites(User $user)
    {
        $includes = 'views,votes,comments';
        $excludes = 'content,tags';
        $comment = DB::table('comments')->join('favorites', 'comments.id', '=', 'favorites.comment_id')->where('favorites.user_id', '=', $user->id)->get();
        
        return $this->prepare($comment);
    }

    /**
     * @param Request $request
     * @param Comment $comment
     *
     * @return array
     */
    public function update(Request $request, Comment $comment)
    {
        $this->validate($request, ['content' => 'required|min:1']);

        $comment->content = $request->get('content');
        if ($request->has('enabled'))
        $comment->enabled = $request->get('enabled');
        if ($request->has('draft'))
        $comment->draft = $request->get('draft');
        $comment->save();

        return $this->prepare($comment);
    }

    /**
     * @param Comment $comment
     *
     * @return array
     */
    public function delete(Comment $comment)
    {
        $comment->delete();

        return $this->prepare($comment);
    }

    protected function prepare(Comment $comment)
    {
        return fractal()
            ->item($comment)
            ->transformWith(new CommentTransformer)
            ->parseIncludes($this->getIncludes())
            ->parseExcludes($this->getExcludes())
            ->toArray();
    }
}
