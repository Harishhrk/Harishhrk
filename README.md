
    it('should display reviewer name in "FirstName LastName" format', () => {
      const mockData = [{
        id: '1',
        comment: 'Test comment',
        createdTime: '2024-03-20T10:30:00Z',
        createdBy: {
          firstName: 'John',
          lastName: 'Doe'
        }
      }];

      jest.spyOn(reviewsApiHook, 'useReviewsQuery').mockImplementation(() => ({
        data: mockData,
        isError: false
      } as never));

      const { getByText } = render(
        <Grid>
          <ViewReviewComponent />
        </Grid>
      );

      expect(getByText('John Doe')).toBeInTheDocument();
    });
